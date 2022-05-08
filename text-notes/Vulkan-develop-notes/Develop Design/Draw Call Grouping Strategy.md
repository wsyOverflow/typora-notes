### Mesh 数据的组织方式

在加载资源过程中，按照如下方式组织 mesh 数据。

- 将所有 mesh 数据加载到连续区域的 buffer 中
  - 目前所有 mesh 共享同一连续区域，之后如果支持动态 mesh 后，可能会根据 mesh 类型划分
- 对于使用某材质的 mesh，可能在场景中以不同变换多次绘制
  - 构建 scene graph 层级结构，用来表示整个场景的逻辑结构，实际的变换由 scene graph 中的层级节点表示
  - 同一 mesh 可以被多个层级节点引用，则表示在场景中多次绘制，每次绘制的 model to world 变换即为引用节点的变换
- mesh 的数据结构，mesh 的数据结构不直接包含其 mesh 数据，而是保存其 mesh 数据在 buffer 区域中的位置描述
  - buffer location: mesh 数据所在 buffer 的起始地址
  - offset: mesh 数据在 buffer 中相对于其实地址的偏移量
  - size: mesh 数据的大小

### 绘制时由 mesh 数据结构到 render proxy

绘制不直接接触 mesh 数据结构，而是使用为每个 render object 构建的 render proxy。在初始化场景渲染信息时，遍历场景结点的 render object 创建 render proxy，并加入场景的 render proxy list 中。一个 render object 可能是一个完整的物体，例如车，而车可能包含多个使用不同材质的 mesh。render proxy 对其拥有的所有 mesh 进行组织，根据材质不同，划分出 meshBatch。每个 meshBatch 保持的是 render object 的一种材质的所有 mesh 信息。因此一个 render object 的 render proxy 包含：

- <mat0, meshBatch0>，<mat1, meshBatch1>，... ...
- 其所属场景结点的 node ID
- proxyIndex：render proxy 在场景的 render proxy list 的索引位置

来自相同的场景结点的 render proxy 可以使用同一个 UBO 传递变换矩阵，来自不同的场景结点。不同场景结点的的render proxy具有不同的变换，需要使用不同的 UBO。而不同材质又需要不同的材质 UBO、不同的 texture 等等。如果按照最直接的 indexed draw call，不同材质、不同场景结点的 render proxy 必须使用不同的 draw call，这会使得 draw call 数量非常多。

### 将大量 render proxy 组织到少量 render group 中

组织策略，使用 SSBO，texture array 减少管线参数的改变频率，使用 indirect draw 减少 draw call 调用带来的开销：

- 对于场景结点变换矩阵等结点数据：使用 SSBO 存储场景结点变换数据，借助内置变量 gl_InstanceIndex 索引场景 SSBO，找到当前 render proxy 对应的结点数据。
- 对于不同材质的参数、贴图：如果材质可以合并（参数种类相同，贴图格式、大小相同），则进行合并。使用 SSBO 存储材质的参数，借助内置变量 gl_InstanceIndex 索引材质 SSBO。合并多个材质的贴图为 texture array。

使用 gl_InstanceIndex 来索引 SSBO 并找到绘制所需参数，需要准备两个 SSBO：

- `scene SSBO`：整个场景共享一个，存储场景中会被绘制到的所有结点的数据（变换等）。使用 `group instance SSBO` 中的 node index 索引
- `group instance SSBO`：每个 render group 共享一个，存储 meshBatch 对应的 node index、material index。需要使用 instance index 索引。在组织 draw call 参数时，需要调整 gl_BaseInstance 来设置 gl_InstanceIndex。
- `group mat SSBO`：每个 render group 共享一个，贴图在 texture array 中的索引（baseColor，normal，emissive等），以及材质参数。使用 `group instance SSBO` 中的 material index 索引

经过场景中的所有 render proxy 得到很多材质和 meshBatch 的组合，例如 <mat0, meshBatch0>，<mat0, meshBatch1> ，<mat1, meshBatch0>，<mat1, meshBatch2> 等等。依次尝试将每个组合合并到现有 render group 中，如果没有找到可以合并的 render group，则为该组合创建一个新的 render group。对于输入 mat+meshBatch 组合遍历现存的每个 render group，如果同时满足以下条件则可以合并：

- 输入 mat+meshBatch 组合的材质种类是否与当前 render group 的材质种类相同，不相同则不可合并。
- 输入 mat+meshBatch 组合的材质贴图格式、大小等是否与当前 render group 的材质相同，不相同则不可合并。

当一个输入 mat+meshBatch 组合可以合并到目标 render group 时，进行合并操作，为目标 render group 生成一个 `group instance`：

- 材质信息存入目标 render group 的材质列表中。该材质位于 render group 中位置作为索引 `material index`。由于多个材质可能引用同一贴图，因此贴图在 texture array 中的索引与材质索引不同。还需要存储不同贴图的索引，如 baseColor 贴图索引、normal map 索引。以及存储材质参数，如 base color 因子等。这些材质数据都存储在 `group mat SSBO` 的一个元素中，如下例结构体：

  ```c++
  struct MaterialUniformBufferParams
  {
      Vector4f BaseColorFactor;
      float RoughnessFactor;
      float MetallicFactor;
      float OcculusionStrength;
  
      uint32_t BaseColorTexIndex;
      uint32_t OccuRoughMetalTexIndex;
      uint32_t NormalMapIndex;
      uint32_t EmissiveMapIndex;
  };
  ```

- meshBatch 信息存入目标 render group。如果已有相同的 meshBatch 则为对应 group instance 增加一个 instance 数据，即输入的 meshBatch 对应的 `{node index、material index}`。如果不存在相同的 meshBatch，则为输入的 meshBatch 新增一个 group instance。这些信息都存储于 `group instance SSBO` 的一个元素中，如下例结构体：

  ```c++
  struct MeshInstanceData
  {
      uint32_t NodeIndexInScene;
      uint32_t MatIndex;
  };
  ```

- render group 的 group instance 创建完成，组织 draw call 信息，得到有效的 `gl_InstanceIndex` ，用以索引 `group instance SSBO`。初始化 instanceOffset 为 0，按照 meshBatch 在 render group 中的顺序，依次进行：

  - 当前 meshBatch 的 group instance 的数量作为 instanceCount，当前的 instanceOffset 作为 firstInstance
  - instanceOffset = instanceOffset+instanceCount

经过以上步骤，可以通过 `gl_InstanceIndex` 索引 `group instance SSBO`，得到当前绘制 mesh 对应的 node index 和 material index。node index 用来索引 `scene SSBO` 获取当前 mesh 对应的场景 node 数据，如 model to world 变换。material index 用来索引`group mat SSBO` 得到当前 mesh 对应的材质参数，以及材质贴图在 texture array 中的索引。



