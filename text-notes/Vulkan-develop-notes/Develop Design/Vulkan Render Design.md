### Summary

本文介绍了本渲染器的 Vulkan 管线管理策略以及 Shader 管理策略。

### Vulkan 管线管理策略

Vulkan 包含三种管线 graphics、compute、raytracing，管线只是一种状态描述，描述了整个工作流中涉及的状态。人们经常提 OpenGL 是个状态机，OpenGL 的状态是由根据使用者的 API 调用来进行切换的，属于 high-level 的 API 设计。而Vulkan 使用 **PSO(pipeline state object)** 来描述管线状态，类型为 `VkPipeline`，这个类型的对象需要用户自己创建。

PSO 只是描述管线的各种状态，状态相同的管线可以使用同一个 PSO，不相容的状态则需要创建新的 PSO。因此对于 PSO 的管理采用将当前状态进行 hash 处理，如果存在当前状态 hash 的 PSO 则直接使用，否则创建新的 PSO 并记录到 unordered_map 中。这部分工作由类 `VulkanPipelineStateManager` 完成，目前的 hash 算法只是简单的移位与异或，留待之后更改。

创建 PSO 的 API 有三个，[`vkCreateGraphicsPipelines`](https://www.khronos.org/registry/vulkan/specs/1.3-extensions/man/html/vkCreateGraphicsPipelines.html) 创建 graphics PSO，[`vkCreateComputePipelines`](https://www.khronos.org/registry/vulkan/specs/1.3-extensions/man/html/vkCreateComputePipelines.html) 创建 compute PSO， [`vkCreateRayTracingPipelinesKHR`](https://www.khronos.org/registry/vulkan/specs/1.3-extensions/man/html/vkCreateRayTracingPipelinesKHR.html) 创建 raytracing PSO。创建 PSO 需要传入描述管线状态的结构体，管线状态的 hash 也主要对这些结构体中的字段进行。另一个 cache 参数，是允许用户将执行过程中创建的 PSO 保存到文件，然后下次启动加载到 cache 参数中，Vulkan 可以通过 cache 来减少创建过程中需要的底层开销，可以提高运行时帧数的稳定性。

对于 PSO 的封装有类 `VulkanPipeline` 及其三个子类 `VulkanGraphicsPipelineState`、`VulkanComputePipeline` 以及 `VulkanRayTracingPipeline`。下面以 graphics PSO 为例，介绍管线的创建流程。

### Graphics PSO 的创建

#### 1. Vulkan Graphics Pipeline 介绍

创建 Vulkan Graphics PSO 的描述封装在结构体 [`VkGraphicsPipelineCreateInfo `](https://www.khronos.org/registry/vulkan/specs/1.3-extensions/man/html/VkGraphicsPipelineCreateInfo.html) 中，下面简要介绍它的成员：

- `VkPipelineShaderStageCreateInfo*` ：管线使用的 shader 信息数组，每个元素包含使用的 shader 对象(VkShaderModule)、入口函数、shader 阶段(vertex/fragment ...)等。
- `VkPipelineVertexInputStateCreateInfo*`：每个元素描述顶点包含的输入属性(vertex shader)，即 location、binding、数据格式以及相对于顶点起始位置的偏移量。
- `VkPipelineInputAssemblyStateCreateInfo`：描述图元数据的拓扑结构(VkPrimitiveTopology)，如       
   POINT_LIST、TRIANGLE_LIST 等。
- `VkPipelineTessellationStateCreateInfo`：曲面细分阶段的配置参数，例如控制点数量
- `VkPipelineViewportStateCreateInfo`：用来描述当前渲染的 viewport 和 scissor 的区域，视口和裁剪可能变化较频繁，可以设置为 dynamic state，创建管线时不设置，在渲染前使用 command 动态设置。
- `VkPipelineRasterizationStateCreateInfo`：描述光栅化阶段的配置参数，几何绘制模式(填充/线框)，正面的顶点环绕方向，剔除面(正面剔除还是背面剔除)。是否开启 depth bias，对 depth buffer 中的深度加上一个偏移量，可以在避免自遮挡时使用
- `VkPipelineMultisampleStateCreateInfo`：描述光栅化阶段的采样数，1 表示无超采样。
- `VkPipelineDepthStencilStateCreateInfo`：描述是否开启深度测试、模板测试，以及深度测试的操作、模板测试的操作。
- `VkPipelineColorBlendStateCreateInfo`：描述 frame buffer 中每个 render target 的 blend 操作，包含 blend 参数、blend 计算方式等。
- `VkPipelineDynamicStateCreateInfo`：描述管线状态中哪些是可以使用 command 指定的动态状态，其他则是创建管线时就确定的。
- `VkPipelineLayout`：包含管线中着色器的输入资源描述
- `VkRenderPass`：管线所使用的 render pass，包含了 render pass 的 render target 的描述。

##### 1.1 Vulkan Render Pass

Vulkan Render Pass 也是一种描述，因此同样使用 hash 方式进行自动创建，主要包含三部分：

- 描述其使用到的所有 attachment 的描述 `VkAttachmentDescription`，一个 attachment 即为一个 image view，用于访问 vulkan texture。进入 renderpass 之前、renderpass 之内、renderpass 结束时的 image layout；加载与存储时的操作；attachment 格式，采样数。
- 描述其内部的 subpass，并且至少包含一个 subpass。每个 subpass 描述包含其 color/resolve/depth attachment 的引用描述，即 framebuffer 中 attachment index、image layout
- subpass 之间的依赖

Vulkan Render Pass 特殊之处在于其是由 subpass 来描述内部多个子阶段的，绘制指令也会被记录在一个当前激活的 subpass 中。subpass 的设计是为了能够高效利用某些 tile-based 架构的移动端 GPU，即 fragment shader 的执行是以 tile 为单位的，因此之后的 subpass 可以直接读到上一个 subpass 写入 attachment 的数据，而不需要等到整个 render pass 执行完。

[[1]](#[1]) 以由两个 pass 组成的 deferred shading 为例，第一个 pass 为了绘制 G-Buffer，第二个 pass 基于 G-Buffer 进行光照计算，得到最终绘制结果。可以设置 render pass 包含两个 subpass，第一个 subpass 绘制 G-Buffer，第二个 subpass 将第一个 subpass 的 G-Buffer 作为 `input attachment`，执行光照计算。流程如下述伪代码：

```c++
cmdBeginRenderPass
	cmdBindPipeline(pipeline0)
	... // bind vertex/index buffer or descriptor set of shader resource
	cmdDraw
	cmdNextSubpass
	cmdBindPipeline(pipeline1)
	... // bind vertex/index buffer or descriptor set of shader resource
	cmdDraw
cmdEndRenderPass
```

注意两个 subpass 使用的是两个 PSO，因为 geometry pass 的 shader 与 deferred shading 的 shader，以及其 shader 输入肯定不同。在第二个 subpass 使用到的 pipeline1 的 shader 则可以使用如下方式来声明 input attachment，

```glsl
layout (input_attachment_index = 0, set = 0, binding = 0) uniform subpassInput inputColor;
layout (input_attachment_index = 1, set = 1, binding = 1) uniform subpassInput inputDepth;
```

`subpassLoad(inputColor)` 则会读取上个 subpass 在 inputColor 同样区域写入的数据。

##### 1.2 Vulkan Pipeline Layout

shader 中的输入参数需要对应描述符 `VkDescriptorSetLayout` 对其描述，而 vulkan pipeline layout 则包含了整个管线的所有 shader 输入参数的描述符。上述 set=0、set=1 则分别表示了索引 0、1 的 VkDescriptorSetLayout 对象，相当于整个管线的描述符分到了两个描述符堆上。一个输入资源的描述符 `VkDescriptorSetLayoutBinding`，包含了 binding、资源类型(如上例中的 `VK_DESCRIPTOR_TYPE_INPUT_ATTACHMENT`)、当为数组时的数组长度以及被哪些 shader 阶段使用。

Vulkan Pipeline Layout 只是对管线 shader 输入的描述，相当于函数签名中形参，真正使用的数据资源，如 sampler、texture、buffer 等，需要使用 command 在 draw call 之前指定。因此 Vulkan Pipeline Layout 同样使用 hash 自动创建。

特别注意，描述符堆是从 descriptorPool 中使用 VkDescriptorSetLayout 创建得到的，每个描述符堆 `VkDescriptorSet` 同时只能由一个 command buffer 使用，因此描述符堆的申请使用与停止使用应该由 command buffer 完成。类 `VulkanDescriptorPoolManager` 维护已经创建的描述符堆以及其使用状态，command buffer 在更新描述符资源之前请求使用，如果没有可用的 descriptorSet 则新创建。command buffer 具有自己执行是否完成的 fence，类 `VulkanCommandBufferManager` 更新 command buffer 状态时，如果 command buffer 中的指令已经完成，则归还其使用的描述符堆。

#### 2. 从上层到渲染层的管线创建

本渲染器的最初设计是面向多平台的，设置了平台无关层，只是目前只有 Vulkan 的渲染层，因此平台无关层更偏向于 Vulkan。下面介绍与管线创建相关的平台无关层的类。

- `GPIRenderPassInfo`：对标 vulkan render pass 的描述，但不仅仅包含了对 render pass 的描述，还包含 render pass 中使用到的 render target ，因此 render pass 对应的 frame buffer 的创建所需参数也是从该类型中获取。
- `GPIGfxPipelineStateInitializer`：对标 Graphics PSO 的配置参数。

抽象类 `GraphicsBridge` 定义了一系列到渲染层的接口，不同的平台实现其接口。例如，设置 render pass 以及管线的接口 `GPIBeginRenderPass` 、`GPISetGfxPipelineState`。下面是管线创建的上层逻辑的伪代码：

```c++
GPIRenderPassInfo basePassInfo;
// ... // config basePass info
gGfxBridge->GPIBeginRenderPass(basePassInfo);
{
    GPIGfxPipelineStateInitializer PSOInitializer;
    // ... // config PSOInitializer
    gGfxBridge->GPISetGfxPipelineState(PSOInitializer, basePassInfo);
    // ... // config the resources of descriptor set
    gGfxBridge->GPIDrawPrimitive();
}
gGfxBridge->GPIEndRenderPass();
```

`GPIBeginRenderPass` 会根据参数 basePassInfo 来生成 vulkan render pass 的配置，以及 frame buffer 的资源参数，通过 hash 的方式查找是否存在对应配置的 render pass 与 frame buffer，不存在则创建。同理 `GPISetGfxPipelineState` 根据 PSOInitializer 与 basePassInfo 参数以 hash 方式查找 PSO，并绑定到当前 command buffer。

同时 `GPIBeginRenderPass` 还做了一些开始管线前的准备工作，比如向 `VulkanCommandBufferManager` 申请 command buffer，以及将 render pass 需要的 render target 调整到其所指定的输入 layout 等。

### Shader 的管理与配置

上述管线创建过程中，还需要 shader 的配置。`UltraShader` 基类定义了 shader 相关描述，如 shader 源码文件路径、shader 编译相关配置、shader 输入参数描述等。不同 shader 继承自此基类，声明自己的参数，其中输入参数会被用于管线创建 `VkPipelineLayout` 资源描述。而编译相关配置则包括一些全局编译配置，如平台、优化参数等，以及一些宏定义，用以开启或关闭某些代码。

##### 1. Shader 源码编译与加载

本渲染器的 shader 编译与加载由类 `ShaderCompilerManager` 完成。该类基于 shaderc，但为了在 renderdoc 能够源码级调试 shader，在 debug 时使用 glslangValidator 编译。目前编译触发条件，有两种：

- 加载过程中，shader 文件及其 include 文件的时间戳要比记录文件中的更新。或者 shader 的一些配置发生改变。
- 运行过程中，shader 的一些配置(如宏定义)发生改变。[TODO] 运行过程中文件的改动主动触发编译

Shader 文件的后缀分为作为头文件的 .glsl 以及表示不同阶段的源码文件如 .ver、.frag、.comp 等。编译过程中会记录源码文件以及其 include 文件的时间戳，用以辨别是否有更改。同时将 shader 编译配置进行 hash，作为编译输出文件的文件名一部分，用以辨别当前 shader 编译配置是否已经编译过。这些信息都会记录在 "intermediate/Shaders/ ShaderFileCache.json" 文件中，该文件在启动时加载。

由于 include 文件会有多级 include，因此只有加载了 shader 源码将所有层级的 include 文件进行时间戳比对才能确定是否需要编译此 shader，因此这一模式只能在加载过程中应用。在运行时需要采用主动触发编译，目前主动触发编译只有实现了 shader 的编译配置发生改变，文件发生改变主动触发编译留待之后实现。

##### 2. Shader 输入参数描述

模板类 `TShaderResourceParameter` 可以容纳各种类型的 shader 参数描述，同时可以绑定实际的资源。但这些目前都需要大量的手写代码完成，手动与 shader 代码里的声明对齐，手动绑定资源。更合理的做法应该是，

- 实现 C++ 反射宏，使用宏能够更快速声明 shader 参数描述
- shader 编译过程应该利用 shader 反射生成包含输入资源描述的头信息。shader 参数在 C++ 中使用相同的名称，使用头信息中的名称进行自动配置参数描述(layout信息等)，并进行验证。

这些留待之后实现。

### TODO

由于时间有限，有很多搁置的策略未实现。

- 描述符堆目前未实现同时多个堆策略
- shader 编译流程生成 layout 头信息，使用 C++ 反射宏进行绑定
- 多线程渲染

### Reference

<a name="[1]">[1]</a> https://www.saschawillems.de/blog/2018/07/19/vulkan-input-attachments-and-sub-passes/















