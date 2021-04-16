# Ray tracing

## Motivation

Ray tracing 的提出是为了解决光栅化的一些缺点，例如光栅化难以处理好全局效果：

- 软阴影（点光源只能做硬阴影）
- Glossy反射：有高光并且粗糙表面
- 间接光照：光线多次弹射

## 光线的一些特性

1. 光线沿直线传播（暂且无视其波动性）
2. 光线之间即使交叉也不会发生碰撞
3. 光线由光源传播到眼睛，也可看做眼睛发出感知光线到光源（可逆性）

## Ray Casting

Key idea: 眼睛到像素、像素到光源，这两个路径若畅通，则该像素可见，否则该像素在阴影中。

<img src=".\Ray tracing.assets\image-20210323235442926.png" alt="image-20210323235442926" style="zoom:25%;" />

<img src=".\Ray tracing.assets\image-20210324105942787.png" alt="image-20210324105942787" style="zoom:25%;" />

## 古老的 Whitted-Style Ray Tracing

有多次弹射，每个可见的弹射点都计算着色，最终所有着色累加到着色像素。图示为 Whitted-Style Ray Tracing 过程。

<img src=".\Ray tracing.assets\image-20210324110420940.png" alt="image-20210324110420940" style="zoom:25%;" />

接下来就是求光线与各种面的交点。

### Ray-Surface Intersection(求交点)

光线的定义：起点(position) $O$, 方向 $\textbf{d}$ (eye到所着色像素的方向)。由此可得到

Ray equation: $r(t)=O+t\textbf{d} \quad 0\leq t\leq \infty$ , 该式定义的是光线上的任一点。

1. Ray intersection with Sphere

球面任一点 $P$ : $(P-C)^2-R^2=0$ , 其中 C 为球心，R 为球半径。

代入光线方程即可求解交点：$(O+t\textbf{d}-C)^2-R^2=0$ , 得交点 $t$。

推广至光线与一般隐式表面求交，隐式表面：$f(P)=0$ , 代入光线方程有 $f(O+t\textbf{d})=0$ 

2. Intersection with Triangle Mesh

如何计算交点？简单想法：与每个三角形求交点，从得到的所有 $t$ 中返回最小的 $t$，即为交点。注意，图形学中忽略光线与三角形共面情况，即只会有 0 或 1 个交点。

Key idea：三角形确定一个平面，可以先求光线与该平面的交点，再判断交点是否在三角形内部。

定义平面任一点$P$ ： $(P-P')\cdot\textbf{N}=0$，其中 $P'$ 是平面上一点，$\textbf{N}$ 是平面法线。代入光线方程解交点，$$(P-P')\cdot \textbf{N}=(O+t\textbf{d}-P')\cdot \textbf{N}=0 \quad=>\quad t=\frac{(P'-O)\cdot \textbf{N}}{\textbf{d}\cdot \textbf{N}}\qquad (0\leq t \leq \infty)$$ 

### Moller Trumbove Algorithm: 使用重心坐标一次解得光线与一个三角形的交点

$$O+t\textbf{d}=(1-b_1-b_2)P_0+b_1P_1+b_2P_2$$，使用线性代数中的克拉姆法则可得

$$\begin{pmatrix} t \\ b_1\\b_2 \end{pmatrix}=\frac{1}{\mathbf{S_1}\cdot\mathbf{E_1}}\begin{pmatrix}\mathbf{S_2}\cdot \mathbf{E_2} \\ \mathbf{S_1}\cdot \mathbf{S} \\ \mathbf{S_2}\cdot \mathbf{d}\end{pmatrix}, where, \begin{cases} \mathbf{E_1} = P_1-P_0 \\ \mathbf{E_2} = P_2-P_0 \\ \mathbf{S}=O-P_0 \\ \mathbf{S_1}=\mathbf{d} \times \mathbf{E_2} \\ \mathbf{S_2}=\mathbf{S} \times \mathbf{E_1} \end{cases}$$ 

如果 $t, b_1,b_2,1-b_1-b_2$都非负，则证明交点在三角形内。

### 计算光线方向 $\bf{d}$

Recall: 光线方向为相机/眼睛到像素的方向，然后以此方向与场景中的三角形面相交。这里注意，场景中的三角形位于world space或者view/camera space中，而像素位于screen/rasterization space中。因此，求光线方向之前，首先将像素坐标转为世界空间下的坐标或者相机空间下的坐标（视场景中的物体坐标是位于世界坐标系还是相机坐标系而定）。

假设有 $width\times height$ 的屏幕，使用光线追踪将场景光栅化到一张 $width\times height$ 的图像上，已经场景中的物体已经位于view/camera space。相机位于原点，朝向 $-Z$-axis，向上为 $+Y$-axis。求解光线方向 $\mathbf{d}$。

 首先我们可以遍历每个像素，为每个像素使用光线追踪进行着色。因为我们要渲染到图像上，所以像素坐标在图像坐标系(即未归一化的 UV 坐标系)，像素对应图像中的坐标为 $(image_x,image_y)$。坐标先从图像坐标系转换到屏幕空间的坐标为 $(screen_x,screen_y)$，光线方向的坐标系要与场景中的物体保持一致，那么需要将 screen space 的坐标 $(screen_x,screen_y)$ 先转换为 clip space的坐标 $(clip_x,clip_y)$ ，再到 view space 的坐标 $(view_x,view_y)$。

<img src=".\Ray tracing.assets\image-20210324144344192.png" alt="image-20210324144344192" style="zoom:30%;" />

image space -> screen space: $\begin{cases} screen_x=image_x \\ screen_y = -image_y \end{cases}$

注意：image的坐标使用像素索引 (i, j) 表示时，要取像素中心 (i+0.5, j+0.5)。

screen space -> clip space: $\begin{cases} clip_x=\frac{2screen_x}{width}-1 \\ clip_y=\frac{2screen_y}{height}-1 \end{cases}=\begin{cases}\frac{2image_x}{width}-1 \\ 1-\frac{2image_y}{height}\end{cases}$ 。

下面从 clip space -> view space，看起来像是透视投影变换的逆变换，但我们并不需要完整地做这样的逆变换。因为，对于 frustum 的视锥，近平面是最终渲染的平面，也成为 image plane。在投影变换中，近平面上的点，$Z$ 坐标不变，$x、y$ 坐标也只受到缩放。下图为 view space 下的 frustum:

<img src=".\Ray tracing.assets\image-20210324145913204.png" alt="image-20210324145913204" style="zoom:30%;" />

对于屏幕中的点，clip space -> view space 的过程相当于，$[-1,1]\times [-1,1]$ 到 $[l.r]\times [b,t]$，对于矩形近平面为 $[-r.r]\times [-t,t]$，矩形宽为 $r$，高为 $t$，宽高比 $aspect \space ratio=\frac{r}{t}$。

$$\begin{cases} view_x= clip_x \cdot r=clip_x \cdot aspect\space ratio \cdot t\\ view_y =clip_y\cdot t \end{cases} = \begin{cases} (\frac{2image_x}{width}-1)\cdot aspect\space ratio\cdot t \\ (1-\frac{2image_y}{height}) \cdot t \end{cases}$$

此时，我们已经得到像素转换到 view sapce 后的 $x,y$ 坐标，未知参数 $t$ 在之后的计算可以消去。下面计算 $z$ 坐标，此时使用垂直方向的 $fov$ 参数。有

$$tan(\frac{fov}{2})=\frac{t}{-view_z}\quad => \quad view_z=-\frac{t}{tan(\frac{fov}{2})}$$

至此，$x,y,z$坐标都已经得到，相机/眼睛位于原点，那么光线方向 $\mathbf{d}=(view_x,view_y,view_z)$.normalized()。因为方向需要 normalize，所以 $x,y,z$ 中的 $t$ 都可以舍去。最终

$$\mathbf{d}=\left((\frac{2image_x}{width}-1)\cdot aspect\space ratio\cdot t, (1-\frac{2image_y}{height}) \cdot t, -\frac{t}{tan(\frac{fov}{2})}\right) \\ =\left((\frac{2image_x}{width}-1)\cdot aspect\space ratio, (1-\frac{2image_y}{height}), -\frac{1}{tan(\frac{fov}{2})}\right).normalized() \\ =\left((\frac{2image_x}{width}-1)\cdot aspect\space ratio\cdot tan(\frac{fov}{2}), (1-\frac{2image_y}{height})\cdot tan(\frac{fov}{2}), -1\right).normalized()$$

至此，如果要为每个像素进行着色，那么一个直接的想法是，遍历所有像素，为每个像素生成一条光线。再求这条光线与场景中所有三角面的交点，取交点中离相机最近的一个。可以看出，这种做法时间复杂度非常高。下面介绍光线追踪前的加速预处理方法。

## Accelerating Ray-Surface Intersection

上述简单求交点方法需要求与每一个物体/三角形的交点，交点中最接近相机的即为所求交点。这种方法的复杂很高 $(pixels \times objects \times bounces)$ 。

### Bouding Volumes (包围盒)

避免求交点的快速方法：将一个复杂的物体包在简单的盒子里。

- 物体完全包含在包围盒中
- 如果光线没有 hit 到包围盒，那么也不会 hit 到物体上
- 因此先做 BVOL，再做物体--光线碰撞测试

<img src=".\Ray tracing.assets\image-20210325163730801.png" alt="image-20210325163730801" style="zoom:30%;" />

三维包围盒通常使用 **Axis-Aligned Boundary Box (AABB)** 轴对齐包围盒，即每对平面都对应一个坐标轴与其像垂直。

y

