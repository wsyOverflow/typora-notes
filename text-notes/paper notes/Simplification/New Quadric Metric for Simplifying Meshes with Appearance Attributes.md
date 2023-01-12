### Summary

QEM 网格简化算法误差计算是采用点到平面的二次距离：对于仅位置坐标的简化，计算顶点坐标到邻接平面的距离，即在 $\mathbf{R}^3$ 空间计算到邻接平面投影点的二次距离；对于带属性的简化，直接从 $\mathbf{R}^3$ 扩展到 $\mathbf{R}^{3+m}$，计算到 $\mathbf{R}^{3+m}$  超平面投影点的二次距离。这样的超平面投影点可分为两部分，三维坐标的投影部分与 $m$ 维属性的投影部分，但此时 $\mathbf{R}^{3+m}$ 空间下的三维坐标投影部分，在 $\mathbf{R}^3$ 空间下的并非是三维坐标投影点。这样会导致属性误差的减小而带来几何误差的增大。

本文提出 New QEM 的误差计算都是在 $\mathbf{R}^3$ 空间进行的。对于三维坐标的误差，沿用 QEM 中仅位置坐标情况下的二次距离计算；对于属性的误差，通过线性方程得出顶点属性与顶点三维坐标的线性关系，基于该线性关系，将属性误差计算同样转换到 $\mathbf{R}^3$ 空间中进行。

### Background

#### 1. Original QEM

##### 1.1 误差与边收缩操作

最初的 QEM 仅计算顶点坐标的二次型误差：点 $\mathbf{v}=(\mathbf{p})\in \mathbf{R}^3$ 到原网格一表面 $f$ 所在平面的二次距离记为 $Q^f(\mathbf{v})$。在简化前，原网格中的每个顶点 $\mathcal{V}$ 先关联与它邻接的表面集合，记为 $f\ni \mathcal{V}$。一个顶点的二次型误差为其到每个邻接表面所在平面的二次距离的加权和，权重为邻接表面的面积，如下：
$$
Q^{\mathcal{V}}(\mathbf{v})=\sum\limits_{f\ni \mathcal{V}}area(f) \cdot Q^f(\mathbf{v}) \label{vertex-quadric} \tag{1}
$$
边收缩操作 $(\mathcal{V}_1,\mathcal{V}_2)\rightarrow \mathcal{V}$，生成新顶点 $\mathcal{V}$，经过最小化新生成顶点的几何二次误差为 $Q^{\mathcal{V}}(\mathbf{v})=Q^{\mathcal{V}_1}(\mathbf{v})+Q^{\mathcal{V}_2}(\mathbf{v})$，即可得到新顶点的位置坐标 $\mathbf{v}$。该最小几何二次误差即作为该边收缩操作引入的误差，按此方法计算所有边收缩操作的误差，之后执行所有边收缩操作中的最小误差者，即完成了一次迭代。

##### 1.2 计算几何二次误差

给定表面 $f=(\mathcal{V}_1,\mathcal{V}_2,\mathcal{V}_3)$，其所在平面 $P\in \mathbf{R}^3$ 的平面方程为 $\mathbf{n}^T\mathbf{p}+d=0$，其中表面法向量
$\mathbf{n}=(\mathbf{p}_2-\mathbf{p}_1)\times (\mathbf{p}_3-\mathbf{p}_1)/||(\mathbf{p}_2-\mathbf{p}_1)\times (\mathbf{p}_3-\mathbf{p}_1)||$，$d=-\mathbf{n}^T\mathbf{p}_1$。平面的法向量 $\mathbf{n}$ 与 $d$ 也可以通过求解下述线性方程组得到，
$$
\begin{pmatrix}
\mathbf{p}_1^T & 1 \\
\mathbf{p}_2^T & 1 \\
\mathbf{p}_3^T & 1
\end{pmatrix}
\cdot 
\begin{pmatrix}
\mathbf{n} \\
d
\end{pmatrix}
=
\begin{pmatrix}
0 \\
0 \\
0
\end{pmatrix}
$$
其中约束条件 $||\mathbf{n}||=1$，相当于共 (3-1) +1 = 3 个未知量，上述线性方程组为 (3x4) (4x1) = (3x1)。

有点 $\mathbf{v}=(\mathbf{p})\in \mathbf{R}^3$，其二次型 $Q^f(\mathbf{v})$ 为点 $\mathbf{p}$ 到平面 $P$ 的二次距离，计算如下，
$$
Q^f(\mathbf{v}=(\mathbf{p}))=(\mathbf{n}^T\mathbf{v}+d)^2=\mathbf{v}^T(\mathbf{nn}^T)\mathbf{v}+2d\mathbf{n}^T\mathbf{v}+d^2
$$

> 按照点到平面的距离公式有，$|\mathbf{n}^T\mathbf{v}+d|/\sqrt{||n||^2}$。由于这里的平面法向量已经标准化，因此二次距离为 $(\mathbf{n}^T\mathbf{v}+d)^2$

将二次型转为标准形式 $\mathbf{v}^T\mathbf{A}\mathbf{v}+2\mathbf{b}^T\mathbf{v}+d^2$，$\mathbf{A}$ 为 3x3 对称阵，$\mathbf{b}$ 为三维向量，$d$ 为标量，有
$$
Q^f=(\mathbf{A},\mathbf{b},c)=((\mathbf{n}\mathbf{n}^T),(d\mathbf{n}),d^2)
$$

##### 1.3 最小值求解

每次边收缩操作都需要计算引入最小误差的新顶点坐标 $\mathbf{v}$，几何二次误差的梯度 $\nabla Q^{\mathcal{V}}(\mathbf{v})=2\mathbf{A}\mathbf{v}+2\mathbf{b}$ 为 $0$ 是得到误差最小的顶点坐标 $\mathbf{v}_{min}$。同样可以通过求解下述线性方程组得到
$$
\mathbf{A}\mathbf{v}_{min} = -\mathbf{b} \label{minimize-v} \tag{2}
$$

#### 2. 带属性的 QEM

假设顶点具有 $m$ 维属性 $\mathbf{v}=\begin{pmatrix}\mathbf{p} \\ \mathbf{s}\end{pmatrix}$，几何二次误差由在 $\mathbf{R}^3$ 中到三维平面的二次距离转为在 $\mathbf{R}^{3+m}$ 中到超平面的二次距离。如果 $\mathbf{v}'=\begin{pmatrix}\mathbf{p}' \\ \mathbf{s}'\end{pmatrix}$ 为顶点在超平面上的投影，那么有 
$$
Q^f(\mathbf{v})=||\mathbf{v}-\mathbf{v'}||^2=||\mathbf{p}-\mathbf{p'}||^2+||\mathbf{s}-\mathbf{s'}||^2
$$
但上述在超平面投影的坐标部分 $\mathbf{p}'$ 并不对应在原始 QEM 中三维平面的投影坐标，这可能导致属性误差的减小而带来几何误差的增大。此时的二次型参数中 $\mathbf{A}$ 为 (3+m)x(3+m) 对称阵，$\mathbf{b}$ 为 3+m 维向量，$d$ 为标量。

同时，为了权衡几何误差与属性误差，可以为每个属性引入自定义权重。同时由于几何尺度不同也会影响误差的数值量级，再计算前先将几何变换到单元立方体中。

### New QEM

本文提出在 $\mathbf{R}^3$ 空间计算二次型误差，几何误差与原始 QEM 一致，而属性误差则通过建立属性与三维坐标的线性关系，基于此关系转到 $\mathbf{R}^3$ 空间计算。顶点 $\mathbf{v}=\begin{pmatrix}\mathbf{p} \\ \mathbf{s}\end{pmatrix}$ 相对表面 $f$ 的二次型误差定义如下：
$$
Q^f(\mathbf{v}=\begin{pmatrix}\mathbf{p}\\\mathbf{s}\end{pmatrix})=Q^f_p(\mathbf{v})+\sum\limits_{j=1}^mQ^f_{s_j}(\mathbf{v})
$$

##### 1 几何误差项

几何误差项 $Q^f_p(\mathbf{v})$ 为点 $\mathbf{p}$ 到其在 $f$ 所属平面的投影点 $\mathbf{p'}$ 的二次距离，这与原始 QEM 一致，只是要将参数使用 $0$ 扩充到 $3+m$ 维，如下所示
$$
Q^f_p=(\mathbf{A},\mathbf{b},c)=
\left(
\begin{pmatrix} 
\mathbf{nn}^T_{(3\times 3)} & \mathbf{0}_{(3\times m)} \\
\mathbf{0}_{(m\times 3)} & \mathbf{0}_{(m\times m)}
\end{pmatrix}_{((3+m)\times (3+m))},
\begin{pmatrix}
d\mathbf{n}_{(3\times 1)} \\
\mathbf{0}_{(m\times 1)}
\end{pmatrix}_{((3+m)\times 1)},
d^2
\right)
\label{geometric-quadric} \tag{3}
$$

##### 2. 属性误差项

首先建立属性与三维坐标的线性关系，对于点 $\mathbf{p}\in \mathbf{R}^3$ 的第 $j$ 维属性值为 $\hat{s_j}(\mathbf{p})$，假设有如下关系
$$
\hat{s_j}(\mathbf{p})=\mathbf{g}^T_j\mathbf{p}+d_j \label{correspondence} \tag{4}
$$
点 $\mathbf{p'}$ 为 $\mathbf{p}$ 在平面 $P\subset \mathbf{R}^3$ 的投影点，同时定义 $\hat{s_j}(\mathbf{p})=\hat{s_j}(\mathbf{p'})$，可以得到 $\mathbf{n}^T\mathbf{g}_j=0$。

>由 $\hat{s_j}(\mathbf{p})=\hat{s_j}(\mathbf{p'})$，$\mathbf{g}^T_j\mathbf{p}+d_j=\mathbf{g}^T_j\mathbf{p'}+d_j$ 可得 $\mathbf{g}^T_j\mathbf{p}-\mathbf{g}^T_j\mathbf{p'}=\mathbf{g}^T_j(\mathbf{p}-\mathbf{p'})=0$。
>
>$\mathbf{p'}$ 为 $\mathbf{p}$ 的投影点，因此 $\mathbf{p}-\mathbf{p'}$ 与平面法向量同向或反向，可得 $\mathbf{n}^T\mathbf{g}_j=0$

上述关系联立，有线性方程组
$$
\begin{pmatrix}
\mathbf{p}_1^T & 1 \\
\mathbf{p}_2^T & 1 \\
\mathbf{p}_3^T & 1 \\
\mathbf{n}^T & 0 \\
\end{pmatrix}_{(4\times 4)}
\cdot 
\begin{pmatrix}
\mathbf{g}_j \\
d_j
\end{pmatrix}_{(4\times 1)}
=
\begin{pmatrix}
s_{1,j} \\
s_{2,j} \\
s_{3,j} \\
0
\end{pmatrix}_{(4\times 1)} \label{correspondence-linear-system} \tag{5}
$$
第 $j$ 维属性二次型项为 $Q^f_{s_j}(\mathbf{v})=(\hat{s_j}(\mathbf{p})-s_j)^2=(\mathbf{g}_j^T\mathbf{p}+d_j-s_j)^2$，转换为标准形式可得到二次型参数为，
$$
Q^f_{s_j}=(\mathbf{A},\mathbf{b},c)=
\left(
\begin{pmatrix}
(\mathbf{g}_j\mathbf{g}_j^T)_{(3\times 3)} & \mathbf{0} & -\mathbf{g}_j & \mathbf{0} \\
\mathbf{0} & \mathbf{0} & \mathbf{0} & \mathbf{0} \\
-\mathbf{g}_j^T & \mathbf{0} & 1 & \mathbf{0} \\
\mathbf{0} & \mathbf{0} & \mathbf{0} & \mathbf{0}
\end{pmatrix},
\begin{pmatrix}
d_j\mathbf{g}_j \\
\mathbf{0} \\
-d_j \\
\mathbf{0}
\end{pmatrix},
d_j^2
\right)
\label{Attribute-Quadirc} \tag{6}
$$
$\mathbf{A}$ 中的 $1$ 出现在 $\mathbf{A}_{3+j,3+j}$，$\mathbf{b}$ 中的 $-d_j$ 出现在 $\mathbf{b}_{3+j}$ 。

##### 3. 顶点的完整二次型

将几何二次型项、$m$ 维属性二次型项加起来就得到顶点的二次型，
$$
Q^f=(\mathbf{A},\mathbf{b},c)=
\left(
\begin{pmatrix}
\mathbf{n}\mathbf{n}^T+\sum_j\mathbf{g}_j\mathbf{g}_j^T & -\mathbf{g}_1 \cdots -\mathbf{g}_m \\
-\mathbf{g}_1^T \\
\vdots & I_{(m\times m)}\\
-\mathbf{g}_m^T
\end{pmatrix},
\begin{pmatrix}
d\mathbf{n}+\sum_jd_j\mathbf{g}_j \\
-d_1 \\
\vdots \\
-d_m
\end{pmatrix},
d^2+\sum_jd_j^2
\right)
\label{new-QEM-params} \tag{7}
$$
其中 $I$ 为 $m \times m$ 单位阵。如果需要对每个属性施加权重来控制相对于几何误差来缩放属性误差，$I$ 的对角线为对应属性的权重。

### Attribute Discontinuities

#### 1. Wedge 定义

通常来说，一个顶点被多个面共享，其在不同的面中虽然坐标相同，但属性可能不同，例如立方体一角的法线。同一顶点(相同坐标)在不同的面称为 Corner，引用同一顶点的所有 Corner 的属性可能相同，也可能不同(不连续)。本文使用 Wedge 来描述不连续属性。

<img src="../../../source/images/New Quadric Metric for Simplifying Meshes with Appearance Attributes.assets/image-20220903161701142.png" alt="image-20220903161701142" style="zoom: 50%;" />

如上图所示，观察位于中心的顶点，在其每个邻接面中都有一个 Corner 引用该顶点。其中相同颜色的 Corner 表示其属性相同(连续)，不同颜色则表示属性不同(不连续)。每种颜色代表一个 Wedge，具有一组属性。因此 Corner 由几项定位：表面索引、顶点索引、wedge 索引。而顶点中则存放唯一的坐标，wedge 中存放唯一的属性的索引。

#### 2. 顶点的二次型误差基于 Wedge 重新定义

因此，在考虑顶点的不连续属性情况下，一个顶点被分为 $k\geq 1$ 个 wedges，每个 wedge $\mathrm{w}_i$ 具有属性 $\mathbf{s}^i$。一个顶点的每个 corner 对应一个面，顶点的 wedge 可能对应多个 corner，即对应多个面 $f\ni \mathrm{w}$，因此定义在 wedge $\mathrm{w}$ 上的二次误差为到其每个面的加权和，权重为面积： 
$$
Q^{\mathrm{w}}(\mathbf{v})=\sum\limits_{f\ni \mathrm{w}}area(f)\cdot Q^f(\mathbf{v})
$$
顶点 $\mathcal{V}$ 的二次型误差定义为它的 $k$ 个 wedges 的二次型误差之和，因此 $\eqref{vertex-quadric}$ 可以重新定义为
$$
Q^{\mathcal{V}}(\mathbf{p},\mathbf{s}^1,\cdots,\mathbf{s}^k)=\sum\limits_{i=1}^{k}Q^{\mathrm{w}_i}(\mathbf{p},\mathbf{s}^i)
$$

#### 3. 边收缩操作基于 Wedge 重新定义

由边收缩生成的新顶点的误差定义 $Q^{\mathcal{V}}(\mathbf{v})=Q^{\mathcal{V}_1}(\mathbf{v})+Q^{\mathcal{V}_2}(\mathbf{v})$，同样需要在 wedge 上重新定义。

<img src="../../../source/images/New Quadric Metric for Simplifying Meshes with Appearance Attributes.assets/image-20220903165559712.png" alt="image-20220903165559712" style="zoom:50%;" />

如上图边收缩操作，新生成顶点的二次型误差为，$\mathcal{V}_1$ 与 $\mathcal{V}_2$ 在边收缩后 wedges 上二次误差之和。

### Simplification Enhancement

#### 1. 低内存开销

原始 QEM 计算顶点的二次型误差总是相对于原网格，而本文提出相对于当前简化网格计算二次型误差。既节省内存，效果又更好

#### 2. Volume Preservation

作者在实验中发现，new QEM 算法会使得简化后的网格模型体积缩小。作者为此引入一个线性约束，
$$
\mathbf{g}_{VOL}^T\mathbf{p}+d_{VOL}=0
$$
$\mathbf{p}$ 为边收缩生成的顶点坐标；$\mathbf{g}_{VOL}$ 为边收缩前的简化网格的所有表面法线加权之和，权重为 $1/3$ 的面积大小。使用拉格朗日乘数 $\gamma$ 
$$
\begin{pmatrix}
\mathbf{A} & \mathbf{g}_{VOL} \\
\mathbf{g}_{VOL}^T & \mathbf{0}
\end{pmatrix}
\begin{pmatrix}
\mathbf{v}_{min} \\
\gamma
\end{pmatrix}
=
\begin{pmatrix}
-\mathbf{b}\\
-\mathbf{d}_{VOL}
\end{pmatrix}
$$
当顶点邻接表面区域的高斯曲率为 0 时，上式与 $\eqref{minimize-v}$ 都无法求解。当无法求解时，采用中点。

### Implementation

已知三角形三个顶点，分别具有一个位置与一组属性，$\mathbf{p}_0,\mathbf{s}_0$、$\mathbf{p}_1,\mathbf{s}_1$、$\mathbf{p}_2,\mathbf{s}_2$。

#### 1. 构建几何二次型参数

$\eqref{geometric-quadric}$ 式中几何二次型参数重写如下
$$
Q^f_p=(\mathbf{A},\mathbf{b},c)=
\left(
\begin{pmatrix} 
\mathbf{nn}^T_{(3\times 3)} & \mathbf{0}_{(3\times m)} \\
\mathbf{0}_{(m\times 3)} & \mathbf{0}_{(m\times m)}
\end{pmatrix}_{((3+m)\times (3+m))},
\begin{pmatrix}
d\mathbf{n}_{(3\times 1)} \\
\mathbf{0}_{(m\times 1)}
\end{pmatrix}_{((3+m)\times 1)},
d^2
\right)
$$
通过三个顶点位置构建表面法向量，以及三角形所在平面方程：

- 边向量：$\mathbf{p}_{01}=\mathbf{p}_1-\mathbf{p}_0$，$\mathbf{p}_{02}=\mathbf{p}_2-\mathbf{p}_0$
- 表面法向量：叉乘向量 $\mathbf{N}=\mathbf{p}_{02}\times \mathbf{p}_{01}$，叉乘向量长度 $length=||\mathbf{N}||$，标准化法向量 $\mathbf{n}=\mathbf{N}/length$，三角形面积 $area=0.5\cdot length$
- 三角形所在平面方程的常数项：$dist=-\mathbf{n}^T\cdot \mathbf{p}_0$，$d_2=dist\cdot dist$，$\mathbf{dn}=dist\cdot\mathbf{n}$。因此有平面方程
    $\mathbf{n}^T\cdot \mathbf{p}+dist=0$
- 用于 volume 约束：$\mathbf{n2a}=\mathbf{N}=2\cdot area\cdot \mathbf{n}$，$dv=-\mathbf{N}^T\cdot \mathbf{p}_0=2\cdot area\cdot dist$，对应平面方程
    $\mathbf{N}^T\cdot \mathbf{p}-\mathbf{N}^T\cdot \mathbf{p}_0=0$
- 上式中 $\mathbf{A}$ 矩阵左上角是一个 3x3 的对称阵，向量乘形如 $\begin{pmatrix}x\\y\\z\end{pmatrix}\begin{pmatrix}x&y&z\end{pmatrix}=\begin{pmatrix}x\cdot x & x\cdot y & x\cdot z\\x\cdot y & y\cdot y & y\cdot z\\x\cdot z & y\cdot z & z\cdot z\end{pmatrix}$，因此需要 6 个参数有：
   $$
   \begin{cases}
   nxx=\mathbf{n}.x\cdot \mathbf{n}.x \\
   nyy=\mathbf{n}.y\cdot \mathbf{n}.y \\
   nzz=\mathbf{n}.z\cdot \mathbf{n}.z \\
   nxy=\mathbf{n}.x\cdot \mathbf{n}.y \\
   nxz=\mathbf{n}.x\cdot \mathbf{n}.z \\
   nyz=\mathbf{n}.y\cdot \mathbf{n}.z \\
   \end{cases}
   $$

至此得到参数  $nxx,nyy,nzz,nxy,nxz,nyz,\mathbf{dn},d_2,area$

#### 2. 构建属性二次型参数

$\eqref{Attribute-Quadirc}$ 式中属性 $s_j$ 的二次型参数重写如下
$$
Q^f_{s_j}=(\mathbf{A},\mathbf{b},c)=
\left(
\begin{pmatrix}
(\mathbf{g}_j\mathbf{g}_j^T)_{(3\times 3)} & \mathbf{0} & -\mathbf{g}_j & \mathbf{0} \\
\mathbf{0} & \mathbf{0} & \mathbf{0} & \mathbf{0} \\
-\mathbf{g}_j^T & \mathbf{0} & 1 & \mathbf{0} \\
\mathbf{0} & \mathbf{0} & \mathbf{0} & \mathbf{0}
\end{pmatrix},
\begin{pmatrix}
d_j\mathbf{g}_j \\
\mathbf{0} \\
-d_j \\
\mathbf{0}
\end{pmatrix},
d_j^2
\right)
$$
求解属性与顶点之间的线性关系：

对 $\eqref{correspondence}$、$\eqref{correspondence-linear-system}$ 描述的属性与顶点坐标的关系进行简单变换，有
$$
\hat{s_j}(\mathbf{p}_1)-\hat{s_j}(\mathbf{p}_0)=\mathbf{g}^T_j\mathbf{p}_1-\mathbf{g}^T_j\mathbf{p}_0=\mathbf{g}_j^T(\mathbf{p}_1-\mathbf{p}_0) \\
\hat{s_j}(\mathbf{p}_2)-\hat{s_j}(\mathbf{p}_0)=\mathbf{g}^T_j\mathbf{p}_2-\mathbf{g}^T_j\mathbf{p}_0=\mathbf{g}_j^T(\mathbf{p}_2-\mathbf{p}_0) \\
\mathbf{n}^T\mathbf{g}_j=0
$$
因此 $\eqref{correspondence-linear-system}$ 可变换为
$$
\begin{pmatrix}
\mathbf{p}_1^T-\mathbf{p}_0^T \\
\mathbf{p}_2^T-\mathbf{p}_0^T \\
\mathbf{n}^T
\end{pmatrix}
\mathbf{g}_j =
\begin{pmatrix}
s_{1,j}-s_{0,j} \\
s_{2,j}-s_{0,j} \\
0
\end{pmatrix}
$$
构建上述线性方程组的系数矩阵有 $\begin{pmatrix}
\mathbf{p}_{01}^T \\
\mathbf{p}_{02}^T \\
\mathbf{n}^T
\end{pmatrix}$，针对第 $j$ 维属性求解方程组得到 $\mathbf{g}_j$，再代入 $\eqref{correspondence}$ 可得 $d_j=\mathbf{g}^T_j\mathbf{p}_0-s_{0,j}$

#### 3. 顶点的完整二次型参数

$\eqref{new-QEM-params}$ 式中顶点坐标与属性的二次型参数重写如下
$$
Q^f=(\mathbf{A},\mathbf{b},c)=
\left(
\begin{pmatrix}
\mathbf{n}\mathbf{n}^T+\sum_j\mathbf{g}_j\mathbf{g}_j^T & -\mathbf{g}_1 \cdots -\mathbf{g}_m \\
-\mathbf{g}_1^T \\
\vdots & I_{(m\times m)}\\
-\mathbf{g}_m^T
\end{pmatrix},
\begin{pmatrix}
d\mathbf{n}+\sum_jd_j\mathbf{g}_j \\
-d_1 \\
\vdots \\
-d_m
\end{pmatrix},
d^2+\sum_jd_j^2
\right)
$$
矩阵 $\mathbf{A}$ 左上角对称阵参数更新为
$$
\begin{cases}
nxx=nxx+\sum_j \mathbf{g}_j.x\cdot \mathbf{g}_j.x\\
nyy=nyy+\sum_j \mathbf{g}_j.y\cdot \mathbf{g}_j.y\\
nzz=nzz+\sum_j \mathbf{g}_j.z\cdot \mathbf{g}_j.z \\
nxy=nxy+\sum_j \mathbf{g}_j.x\cdot \mathbf{g}_j.y \\
nxz=nxz+\sum_j \mathbf{g}_j.x\cdot \mathbf{g}_j.z \\
nyz=nyz+\sum_j \mathbf{g}_j.y\cdot \mathbf{g}_j.z \\
\end{cases}
$$
矩阵 $\mathbf{A}$ 右上角对角阵的对角线上的每个元素分别对应一个属性的权重。

$\mathbf{b}$ 中参数 $d\mathbf{n}$ 更新为
$$
d\mathbf{n}=d\mathbf{n}+\sum_jd_j\mathbf{g}_j
$$
$c$ 参数更新为
$$
d2=d2+\sum_jd_j^2
$$


至此得到上式中所有二次型参数，共 11+5m 个参数

- $nxx,nyy,nzz,nxy,nxz,nyz,d\mathbf{n},d_2,area$，6+3+1+1=11 个
- $\mathbf{g}_1,\cdots,\mathbf{g}_m$，mx3=3m 个
- $d_1,\cdots,d_m$，m 个
- 属性权重 m 个

#### 4. 计算收缩操作误差

已知边 $\{\mathbf{p}_0,\mathbf{p}_1\}$，表面法向量 $\mathbf{n}$，计算以下参数

- 边 $\mathbf{p}_{01}=\mathbf{p}_1-\mathbf{p}_0$ 与 $\mathbf{n}$ 叉乘向量：$\mathbf{NE}=\mathbf{p}_{01}\times \mathbf{n}$，长度 $nelen=||\mathbf{NE}||$，标准化 $\mathbf{ne}=||\mathbf{NE}||/nelen$

- 参数 $a=weight\cdot ||\mathbf{p}_{01}||$

- $$
  \begin{cases}
  nexx=a\cdot \mathbf{ne}.x\cdot \mathbf{ne}.x \\
  neyy=a\cdot \mathbf{ne}.y\cdot \mathbf{ne}.y \\
  nezz=a\cdot \mathbf{ne}.z\cdot \mathbf{ne}.z \\
  nexy=a\cdot \mathbf{ne}.x\cdot \mathbf{ne}.y \\
  nexz=a\cdot \mathbf{ne}.x\cdot \mathbf{ne}.z \\
  neyz=a\cdot \mathbf{ne}.y\cdot \mathbf{ne}.z
  \end{cases}
  $$





#### 实现

##### 1. 不带属性的二次型 Quadric

所有二次型参数的存储形式都为乘上面积后的值。

##### 2. 带有属性的二次型 QuadricAttribute

接收三角形三个顶点的位置和属性 $(\mathbf{p}_0,\mathbf{s}_0)$、$(\mathbf{p}_1,\mathbf{s}_1)$、$(\mathbf{p}_2,\mathbf{s}_2)$。

计算属性在三角平面的投影系数 $\mathbf{g}$ 与 $d$，直接累积到二次型。
$$
\begin{cases}
nxx=nxx+\sum_j \mathbf{g}_j.x\cdot \mathbf{g}_j.x\\
nyy=nyy+\sum_j \mathbf{g}_j.y\cdot \mathbf{g}_j.y\\
nzz=nzz+\sum_j \mathbf{g}_j.z\cdot \mathbf{g}_j.z \\
nxy=nxy+\sum_j \mathbf{g}_j.x\cdot \mathbf{g}_j.y \\
nxz=nxz+\sum_j \mathbf{g}_j.x\cdot \mathbf{g}_j.z \\
nyz=nyz+\sum_j \mathbf{g}_j.y\cdot \mathbf{g}_j.z \\
\end{cases}
$$
最后$nxx,...,nyz$，$\mathbf{g}$ $d$ 都会乘上面积权重，所有二次型参数的存储形式都为乘上面积后的值。

##### 3. 二次型优化 QuadricAttrOptimizer

支持累积 Quadric 和 QuadricAttribute 的二次型参数，同时累加面积。





















