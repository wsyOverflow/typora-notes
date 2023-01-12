## Summary

本文提出一种基于 Mesh-Shader 联合简化方法来生成 Mesh 和 Shader 简化变体组合的 LOD。对于 Mesh-Shader 简化变体的优劣使用绝对误差(渲染结果图像的差异指标)和渲染时间开销来评估。对于联合简化算法，论文同时提出了接近全局最优的联合优化算法和高效近似的独立优化算法。这两种算法最终都会为每个 LOD 生成一组满足绝对误差上限限制并且时间开销最优的 Mesh-Shader 简化变体对集合。最后使用图结构来描述 LOD 之间的切换，图中边的权重为切换带来的过渡误差。在这个图上应用最短路径算法即得到最终的 Mesh-Shader LOD。

## Problem

假设原 Mesh 为 $M_0$，原 Shader 为 $S_0$，整个渲染过程记为函数 $f$，$f(M_0,S_0)$ 返回最终渲染图像上每个像素的颜色。LOD $i$ 表示到相机的距离为 $d_i$，联合简化即对于距离 $(d_i,d_{i+1})$ 自动化生成一系列简化 Mesh 和 简化 Shader $(M_i,S_i,d_i)$。

### 1. 质量评估指标(Quality Error Metric)

为了使得简化后的 Mesh-Shader $(M_i,S_i)$ 与原 Mesh-Shader $(M,S)$ 渲染结果差异小且 LOD 切换的过渡差异小，还需要评估 $(M_i,S_i)$ 质量上的优劣。 这里用到的评估指标有 **absolute error**(绝对误差) $\varepsilon_a$ 和 **transition error**(过度误差) $\varepsilon_t$：

- absolute error $\varepsilon_a$ 表示 Mesh-Shader 简化对与原 Mesh-Shader 的差异，如下式在 LOD $i$ 的绝对误差
  $$
  \varepsilon_a(i)=\int_H||f(M_i,S_i,d_i)-\bar{f}(M,S,d_i)||\space dH \tag{1} \label{absolute error}
  $$
  上式中 $\bar{f}$ 表示具有超采样的渲染过程，即绝对误差的评估的基准为带有抗锯齿的渲染结果。

- transition error $\varepsilon_t$ 表示两个相邻 LOD 的简化 Mesh-Shader 的差异，如下式 LOD $i$ 与 LOD $i+1$ 的过渡误差，都渲染在 $d_{i+1}$ 距离
  $$
  \varepsilon_t(i)=\int_H||f(M_i,S_i,d_{i+1})-f(M_{i+1},S_{i+1},d_{i+1})||\space dH \tag{2} \label{transition error}
  $$

**绝对误差** $\varepsilon_a$ 用来评估 LOD $i$ 下所有的 Mesh-Shader 简化对 $(M_i,S_i)$ 与基准渲染结果(原 Mesh-Shader)的差异，从而选出最最接近基准渲染结果的 tuple 集合。**过渡误差** $\varepsilon_t$ 用来评估相邻 LOD 之间过渡是否平滑，从最优的 $(M_i,S_i)$ 集合中选出过渡最平滑(过渡误差最小) 的一组。

### 2. 性能评估指标(Rendering Time)

Mesh-Shader 联合简化的目的是为了降低渲染开销，并且简化后的渲染结果与原 Mesh-Shader 的渲染结果肉眼分辨不出差异，因此在获取最低渲染开销的 $(M_i,S_i)$ 时，还需要限制其误差范围。有如下优化过程：
$$
\mathop{\arg\min}_{M_i,S_i,d_i}\ \ t = Cost\big(f(M_i,S_i,d_i)\big) \quad \mathbf{s.t.} \quad \varepsilon_a(i)<e_a(d_i)\cdot s_{d_i} \tag{3} \label{joint optimization}
$$
其中，$t$ 是在 $d_i$ 距离下渲染 $(M_i,S_i)$ 的时间开销，即性能指标；$e_a(d_i)$ 为在 $d_i$ 距离下的每像素绝对误差上限，上式即在确保 $(M_i,S_i)$ 的绝对误差 $\varepsilon_a(i)$ 不超过这个上限的情况下，选取最小的时间开销。每像素绝对误差上限采用：
$$
e_a(d)=(\frac{d-d_{near}}{d_{far}-d_{near}})^Q\cdot e_{max} \tag{4}\label{absolute error bound}
$$
其中 $e_{max}$ 是每像素最大绝对误差上限；LOD 越大，即物体离相机越远，对应像素能容忍的绝对误差也越大，参数 $Q\in[0,1]$ 则用来决定在 LOD 增大时，质量允许下降的速度。

### 3. LOD 过渡评估指标

在 Mesh-Shader 联合简化完成后，在每个 LOD 上都得到一批简化后的 Mesh-Shader。此时不仅要为每个 LOD 挑选出满足绝对误差上限限制并且时间开销最优的 Mesh-Shader 简化变体对，而且要使得相邻 LOD 切换时，不会带来明显的视觉感知，即过渡平滑。过渡是否平滑需要上述过渡误差 $\varepsilon_t(i)$ 来评估。假设在 LOD $i$ 上得到一批性能和质量最优的 Mesh-Shader 简化对集合 $\{(M_p,S_p)\}_i$，而 LOD $i+1$ 的最优 Mesh-Shader 简化对集合为 $\{(M_q,S_q)\}_{i+1}$，有过渡误差：
$$
\varepsilon_t(i_p,(i+1)_q)=\int_H \ ||f(M_p,S_p,d_{i+1})-f(M_q,S_q,d_{i+1})||\ dH \tag{5}\label{LOD transition error}
$$
上式定义了由 LOD $i$ 的 $(M_p,S_p)_i$ 切换到 LOD $i+1$ 的 $(M_q,S_q)_{i+1}$ 带来的过渡误差。

### 4. Error Model

在实际计算 error 时，为了捕获到物体的不同部分，会渲染出不同观察方向的多张图像。通常不同的观察方向由 Uniform 参数 $U$ 确定，因此在求误差时还需要考虑不同的 $U$，包括观察方向、光照配置。误差计算如下：
$$
\varepsilon_a(i)=\sum\limits_V\int_U \iint_{xy}||f(M_i,S_i,d_i)-\bar{f}(M,S,d_i)||^2\space dxdy \\
\varepsilon_t(i)=\sum\limits_V\int_U \iint_{xy}||f(M_i,S_i,d_{i+1})-f(M_{i+1},S_{i+1},d_{i+1})||^2\space dxdy
$$
可知计算像素误差时采用的是 $L_2$ 范数，并且将不同视角下的误差累计。

## Approach

整个算法包含两个步骤：第一步，Mesh-Shader 联合简化生成不同 LOD 的 Mesh-Shader 简化变体集合；第二步，从这些变体集合中挑选出最优的一组作为最终的 LOD。

### 1. 生成 Mesh-Shader 简化变体

 首先将观察距离范围划分为 $N$ 个层级(LOD)，接下来对每个 LOD 所代表的距离生成 Mesh-Shader 简化变体。这里论文提供两种简化方法，一种是 **Joint Optimization**，另一种是 **Separate Optimization**。

#### 1.1 Joint Optimization

Joint Optimization 方法所解决的优化问题如 $\eqref{joint optimization}$ 式所示，该方法对 Mesh 和 Shader 同时探索简化，下面先分别介绍 Shader 简化方法与 Mesh 简化方法。

##### 1.1.1 Shader Simplification

Shader 简化是基于 genetic programming 的思想，先将 Shader 代码转为 Abstract Syntax Trees (ASTs) 和Program Dependence Graphs (PDGs)，再通过简化规则对 AST 进行修改，最终将修改后的 AST 转回 Shader 源码，即得到 Shader 变体。简化规则详细请查看 [[1]](#[1])、[[2]](#[2])、[[3]](#[3]).

##### 1.1.2 Mesh Simplification

对于 Mesh 简化，论文使用 [[4]](#[4]) 提出的简化框架，只是将其使用的 geometric metric 替换为本论文使用的 image error metric。在简化过程中迭代应用 edge collapse operation，但将一条边缩小为一个顶点的 placement policy 计算量过高，因此使用简单的 place-to-endpoint policy，即直接将顶点放置在具有较低的 image error 的端点处。对于那些在屏幕空间长度大于 $D$ 屏幕像素的边不进行 collapse，论文中的实现采用 $D=1$。

##### 1.1.3 联合简化生成 Mesh-Shader 简化变体 LOD

联合简化过程即先生成一批 Shader 变体，再在其中的 pareto 变体上分别进行 Mesh 简化，得到一批满足绝对误差限制的 Mesh-Shader 简化对，选取性能较好的一批进入下一次简化流程，重复该步骤。如下图所示：

<img src="Automatic Mesh and Shader Level of Detail.assets/image-20220209230120207.png" alt="image-20220209230120207" style="zoom:50%;" />

将原 Shader 作为上图中的 Input Shader，开始对 LOD $i$ 的联合简化过程：

1. Input Shader 先经过 genetic programming  得到第一批(Generation 1) 的 Shader 变体。将第一批的 Shader 变体中的 pareto 变体分别应用于 Mesh 简化流程中，得到绝对误差在每像素绝对误差上限内的所有 Mesh-Shader 简化对。
2. 对生成的简化对进行性能由高到低排序，然后从中挑选出性能前 $N\%$(论文实现中 $N=20$) 的一批 Shader 变体作为 Generation 2 的 Input Shader，继续重复 1 步骤直至满足最大简化深度设定(图中为3)。

达到最大简化深度后即完成了对 LOD $i$ 的联合简化，可以得到 LOD $i$ 的一批 Mesh-Shader 简化对。对所有 LOD 都进行上述步骤，即可得到所有 LOD 的 Mesh-Shader 简化对。

#### 1.2 Separate Optimization

Joint optimization 算法可以接近达到全局最优，但是非常耗时。因此论文提出 Separate Optimization，分别进行 Mesh 和 Shader 简化过程，分别得到一批 Mesh 变体和一批 Shader 变体，再在这两批中挑选出最优的 Mesh-Shader 变体对。虽然说是单独进行 Mesh(Shader) 简化，但在 Mesh(Shader) 简化过程中，同时也考虑到了 Shader(Mesh) 的简化影响。

##### 1.2.1 生成 Mesh 简化候选变体

在生成 Mesh 简化变体时，通过对原 Shader 在不同 LOD 距离下的渲染结果进行超采样来近似简化 Shader，即使用原 Shader 渲染不同 LOD 距离的简化 Mesh 后进行超采样的结果近似为简化 Shader 渲染简化 Mesh 的结果。这种近似方法下，得到的与 reference 的差异相当于考虑了简化 Shader，得到的 Mesh 简化变体即为联合简化得到的 Mesh 简化变体。

##### 1.2.2 生成 Shader 简化候选变体

生成 Shader 简化变体。在本步骤中，上个步骤生成的不同 LOD 下的 Mesh 简化变体作为已知场景设定，本步骤即为生成这些场景设定下的 Shader 简化变体。为了加速，从所有 LOD 中均匀选出 4 个 LOD，再对每个选出的 LOD 中选出 top-10 简化 Mesh，由此得到 40 种场景设定。对于每种设定，都使用 genetic programming 生成一组 pareto shader 变体。在 Shader 简化过程中，对绝对误差和性能的评估并不是通过实际的渲染得到的，而是使用 [[1]](#[1]) 中的预测模型。由于引入了简化 Mesh，为了预测结果更准确，还对该预测模型进行了扩展，使其考虑到顶点和像素数量。

经过本步骤，会生成 40 组 Shader 候选变体，但注意这些变体并不特定在某一 LOD 上，而是针对所有的 LOD。

##### 1.2.3 查找候选变体生成 Mesh-Shader 简化变体 LOD

上述步骤生成了一批 Shader 候选变体和每个 LOD 下的一批 Mesh 候选变体，接下来需要在其中查找每个 LOD 下的最优 Mesh-Shader 简化变体。如果直接进行查找，复杂度会比较高，因此先做排序再查找，可以将 2D 查找问题降为 1D。这里的排序是基于绝对误差和性能的，在上述步骤中使用的都是预测结果，因此本步骤需要得到真是的绝对误差和性能。对假设 Shader 候选变体集合 $\{S_l\}$ 和 LOD $i$ 的 Mesh 候选变体集合 $\{M_k\}_i$ ，在 LOD $i$ 下的排序为：

- 对 Shader 候选变体集合 $\{S_l\}$ 按照绝对误差递增的顺序排序。这里要计算 Shader 变体 $S_l$ 在 LOD $i$ 下的绝对误差，是通过  $[M_0,S_0]$ 和 $[M_0,S_l]$ 之间的差异来衡量的。
- 对于 Mesh 简化变体集合 $\{M_i\}$，$i$ 越大表示简化程度越大，对应的绝对误差增大、性能开销减小。因此无需实际渲染即可得到一个绝对误差递增、性能开销递减的序列。 

排序后的结果如下图所示：

<img src="Automatic Mesh and Shader Level of Detail.assets/image-20220210131107321.png" alt="image-20220210131107321" style="zoom:67%;" /> 
其中纵轴为 Shader 候选变体集合，随着纵坐标的增大，其变体的绝对误差增大，而渲染开销未知。横轴为 Mesh 候选变体集合，随着横坐标增大，简化程度增大，其绝对误差增大、渲染开销减小。在这两组排好序的候选变体中查找最优的 Mesh-Shader 变体对，则可按照上图折线表示的步骤查找。算法先从简化程度最大的 Mesh 和原 Shader 开始，即 $[M_K,S_0]$，$K$ 为当前 LOD 的 Mesh 候选变体总数。假设当前查找的 Mesh-Shader 变体对为 $[M_i,S_j]$，算法步骤如下：

1. 通过实际的渲染评估  $[M_i,S_j]$  的绝对误差与渲染时间。
2. 如果的绝对误差位于误差限制范围内，则 $j=j+1$，并加入到当前 LOD 的最终 Mesh-Shader 变体集合中；否则 $i=i-1$。
3. 重复步骤 1、2，直至查找完边界上所有候选变体。

对当前 LOD 的最终 Mesh-Shader 变体集合按照渲染时间进行排序，至此我们得到了当前 LOD 下的最优 Mesh-Shader 变体集合。查找算法伪代码如下所示：

<img src="Automatic Mesh and Shader Level of Detail.assets/image-20220210131853684.png" alt="image-20220210131853684" style="zoom: 50%;" />

对所有 LOD 都进行上述步骤，则可以得到每个 LOD 的最优 Mesh-Shader 变体集合。

#### 1.3 LOD 间的平滑过渡

 上述两个优化方法都可以得到每个 LOD 下的最优 Mesh-Shader 变体集合，假设为 $\{(M_p,S_q)\}_i$ 。本部分则需要从这几组变体集合中，最终生成过渡最平滑的 LOD；同时，通过删去不必要的 LOD，优化过渡层级。

##### 1.3.1 通过图的最短路径算法的最平滑的 LOD

对于生成的 LOD，还需要保证相邻 LOD 在切换时的过渡平滑，过渡平滑由过渡误差 $\eqref{LOD transition error}$ 来评估。假设  $(M_p,S_p)_i$ 和 $(M_q,S_q)_{i+1}$ 分别为 LOD $i$ 和 LOD $i+1$ 的 Mesh-Shader 简化对。将 $(M_p,S_p)_i$ 和 $(M_q,S_q)_{i+1}$ 看作是一条边的两个顶点，这条边的权重为对应的过渡误差。按照这种方法，可以使用所有 LOD 的 Mesh-Shader 变体以及相邻 LOD 的过渡组成一个图。之后在这个图上应用最短路径算法(Dijkstra’s algorithm)，即可得到 LOD 过渡的一条最短路径，表示生成了具有最小过渡误差的 LOD，这条最短路径上的所有顶点即为最终的 Mesh-Shader LOD。算法伪代码如下图所示：

<img src="Automatic Mesh and Shader Level of Detail.assets/image-20220210135653484.png" alt="image-20220210135653484" style="zoom: 50%;" />

##### 1.3.2 优化过渡层级

如果划分的 LOD 层级过多，会带来较高的内存开销，因此论文通过减少 LOD 层级给出一种 trade off。对于上一步得到的最短路径，假设相邻两个 LOD 的 Mesh-Shader 变体对的时间开销分别为 $t_i$ 和 $t_{i-1}$。如果 $(t_{i-1}-t_i)/t_0<W$，那么说明两个相邻 LOD 的性能非常接近，LOD $i$ 可以被直接丢弃，论文实现中 $W=4\%$ 。因此，这些无效的 LOD 层级被删除，使用 LOD 的内存开销就会降下来。

## Reference

<a name="[1]">[1]</a> Y. He, T. Foley, N. Tatarchuk, and K. Fatahalian, “A system for rapid, automatic shader level-of-detail,” ACM Trans. on Graph. (TOG), vol. 34, no. 6, p. 187, 2015.  

<a name="[2]">[2]</a> R. Wang, X. Yang, Y. Yuan, W. Chen, K. Bala, and H. Bao, “Automatic shader simplification using surface signal approximation,” ACM Trans. on Graph. (TOG), vol. 33, no. 6, p. 226, 2014.  

<a name="[3]">[3]</a> P. Sitthi-Amorn, N. Modly, W. Weimer, and J. Lawrence, “Genetic programming for shader simplification,” in ACM Transactions on Graphics (TOG), vol. 30, no. 6. ACM, 2011, p. 152.  

<a name="[4]">[4]</a> M. Garland and P. S. Heckbert, “Surface simplification using quadric error metrics,” in Proceedings of the 24th annual conference on Computer graphics and interactive techniques. ACM Press/Addison-Wesley Publishing Co., 1997, pp. 209–216.  