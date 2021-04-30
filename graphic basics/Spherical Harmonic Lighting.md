## Summary

SH lighting 引入的是一种预计算技术，使得接近物理的渲染方程 BRDF 可以应用于实时渲染。

## BRDF 

**Rendering Equation:**

$$L(x,\omega_o)=L_e(x,\omega_o)+\int_Sf_r(x,\omega_i\rightarrow\omega_o)L_i(x',\omega_i)G(x,x')V(x,x')$$，其中

- $L(x,\omega_o)$：点 $x$ 沿着 $\omega_o$ 方向发出的光

- $L_e(x,\omega_o)$：物体在点 $x$ 处的自身发光项

- $f_r(x,\omega_i\rightarrow\omega_o)$：点 $x$ 处的 BRDF，描述的是沿 $\omega_i$ 方向的入射光在点 $x$ 处如何转化为沿着 $\omega_o$ 方向的出射光，即分布在 $\omega_o$ 方向的光强度比例
- $L_i(x',\omega_i)$：从发光物体上一点 $x'$ 发出的沿着 $\omega_i$ 方向到 $x$ 的入射光
- $G(x,x')$：$x$ 与 $x'$ 的几何关系，一般指入射方向 $\omega_i$ 与 点 $x$ 处法线 $n$ 之间的夹角
- $V(x,x')$：visibility test，0 表示 $x$ 与 $x'$ 之间被遮挡，1 表示 $x$ 与 $x'$ 之间可达

渲染方程中的积分项使用蒙特卡洛采样估计，若要得到噪声很低的采样结果，需要进行大量的采样，因此计算复杂度过高，难以在实时渲染中直接应用。SH lighting 技术将大量的计算转移到预计算过程，实时渲染过程中只需加载预计算得到的数据并进行少量计算。在进入正题之前，首先介绍一下基础知识。

## Basis Functions(基函数)

基函数是一些小的信号片段，它们可以被缩放和组合，从而产生与原始函数的近似。组合方式为线性组合，那么需要首先求解该线性组合中每个基函数对应的系数，这个过程被称为 **projection(投影)**。投影过程的数学形式为，

$$c_i=\int f(x)B_i(x)dx$$，其中 $f(x)$ 为原函数，$B_i(x)$ 是其中一个基函数，该式可理解为 $f(x)$ 投影在基函数 $B_i(x)$ 的坐标为 $c_i$。下面是使用一组线性基函数的组合来近似 $f(x)$ 的过程。

| 投影过程                                                     | 缩放基函数                                                   | 线性组合基函数 $f(x)\approx\sum c_iB_i$                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| <img src=".\Spherical Harmonic Lighting.assets\image-20210419164032869.png" alt="image-20210419164032869" style="zoom:25%;" /> | <img src=".\Spherical Harmonic Lighting.assets\image-20210419164111329.png" alt="image-20210419164111329" style="zoom:25%;" /> | <img src=".\Spherical Harmonic Lighting.assets\image-20210419164226119.png" alt="image-20210419164226119" style="zoom:25%;" /> |

基函数有多种选择，通常我们会选择一类特殊的基函数——**Orthogonal Basis Functions(正交基函数)**。定义如下，

$$\int B_m(x)B_n(x)dx=\begin{cases} 0 \quad n\neq m \\ c \quad n=m \end{cases}$$，即一个基函数向另一个基函数的投影坐标为 0，向自身投影为一个常数。

更特殊的有，当 $c=1$ 时，这组基为**Orthonormal Basis Functions(规范正交基)**。

#### 1. 1-D Case(Legendre polynomials)

Legendre 多项式是一种正交基，通常是复值函数，我们这里只考虑其实值版本 Associated Legendre 多项式，用 $P$ 表示。Associated Legendre 多项式，

- 取值范围 $[-1,1]$ 
- 两个参数 $l$ 和 $m$，$l$ 表示阶数(band)，$m\in[0,l]$ 表示某一阶的基函数的索引
- 由其正交性，同一阶的基函数投影为一个常数，不同阶投影常数不同

前 6 个基函数可以写为，

$$P^0_0(x) \\ P^0_1(x),P^1_1(x) \\ P^0_2(x),P^1_2(x),P^2_2(x)$$<img src="D:\Study\Paper-reading\graphics\typora-notes\graphic basics\Spherical Harmonic Lighting.assets\image-20210419193538628.png" alt="image-20210419193538628" style="zoom: 50%;" />

Legendre 多项式用于近似 1-D 函数，一般 $P^0_0(x)$ 预先定义好，使用下述递归规则进行推导高阶或同阶基函数

1. $$(l-m)P^m_l=x(2l-1)P^m_{l-1}-(l+m-1)P^m_{l-2}$$
2. $$P^m_m=(-1)^m(2m-1)!!(1-x^2)^{m/2}$$
3. $$P^m_{m+1}=x(2m+1)P^m_m$$

> 这三种规则适用于不同的情况

#### 2. 2-D Case(Spherical Harmonics)

SH 是定义在球面上的基函数，因此是二维(注意在球面，不是在球内，转为球坐标即为二维)，同样这里也只考虑其实值形式。**只考虑球心位于原点的单位球**，将三维空间坐标转为球面坐标有

​				$$(x,y,z)\rightarrow (sin\theta cos\phi,sin\theta sin\phi,cos\theta)$$，

SH 函数由 $y$ 表示，

$$y^m_l(\theta,\phi)=\begin{cases} \sqrt{2}K^m_lcos(m\phi)P^m_l(cos\theta), \quad \quad m>0 \\\sqrt{2}K^m_lsin(-m\phi)P^{-m}_l(cos\theta), \space m<0 \\ K^0_lP^0_l(cos\theta) , \quad \quad \quad \quad \quad \quad \quad  m=0\end{cases}$$，其中 $P$ 是 Associated Legendre 多项式，$K$ 是用于 normalize 的缩放因子，

$$K^m_l=\sqrt{\frac{(2l+1)(l-|m|!)}{4\pi (l+|m|!)}}$$

> 注意这里的 $m$ 与 Associated Legendre 多项式中定义有所不同，$l$ 仍是由 $0$ 开始的正整数，$m$ 取 $[-l,l]$ 的有符号整数。$y^m_l(\theta,\phi) \quad where l\in \mathbf{R}^+,-l\leq m \leq l$

用于索引 SH 基函数需要 $l$ 和 $m$ 两个参数，相当于二维矩阵，有时将其展开为一维向量更方便，

$$y^m_l(\theta,\phi)=y_i(\theta, \phi)$$，where  $i=l(l+1)+m$ 。

<img src=".\Spherical Harmonic Lighting.assets\1.PNG" alt="1" style="zoom: 67%;" />

> 0-band 只是一个正常数，如果你只是用 0-band 系数渲染 self-shadowing 模型，渲染结果看起来像 accessibility shader，其中那些在裂缝(高曲率)深处的点的着色要暗于那些在平滑表面的点。
>
> 仅使用 1-band 的系数的线性组合能够很好的近似 diffuse surface 反射模型中 cosine 项。

##### SH Porjection

将一个定义在球面上的函数投影到 SH 基函数的过程同上述投影一样，计算单个投影得到的系数，

$$c^m_l=\int_S f(S)y^m_l(S)dS \tag{1}$$

按此方式得到每一个基函数对应的系数，然后进行线性组合重构原函数，得到原函数的近似

$\tilde{f}(s)=\sum^{n-1}_{l=0}\sum^l_{m=-l}c^m_ly^m_l(s)=\sum^{n^2}_{i=0}c_iy_i(s)$

可以看出，$n$ 阶近似的复杂度为 $O(n^2)$。

> 当阶数 $n$ 趋向于无穷时，可以重构出真正的原函数 $f(x)$。但实际中只会使用前一部分阶，即 band-limited 近似。阶数越高，频率越高，band-limited 近似就相当于去除高于某个阈值的频率，因此 SH 适用于低频光(diffuse,glossy)，不适用于 specular(会需要非常高的阶)。

**Sample**

定义一个简单的光源方程，

$$light(\theta,\phi)=max(0,5cos\theta-4)+max(0,-4sin(\theta-\pi)*cos(\phi-2.5)-3)$$。

<img src="./Spherical Harmonic Lighting.assets\image-20210422203333538.png" alt="image-20210422203333538" style="zoom: 33%;" />

将 $light(\theta,\phi)$ 投影到 SH 空间即求下述积分，

$$c_i=\int^{2\pi}_0\int^\pi_0light(\theta,\phi)y_i(\theta,\phi)sin\theta d\theta d\phi$$

> 此处积分只是投影公式 (1) 的球坐标形式，其中 $dS=sin\theta d\theta d\phi$，$dS$ 为球面上小矩形的面积，可知越接近极点的小矩形面积越小，越接近赤道的小矩形面积越大。具体推导参考 [微分立体角对应的微分面积](file:./Basic Radiometry.md)。

计算此积分，使用蒙特卡洛采样方法进行估计，此时使用简单的均匀分布的概率密度，由在球面上的积分得 $p=1/4\pi$。

至此，我们得到了光源方程得 SH 近似，即 $light(\theta,\phi)=\sum_ic_iy_i(\theta,\phi)$。

> 以同样得方程将 BRDF 函数投影到同一 SH 空间，由 SH 基函数的规范正交性即可将 rendering equation 从积分降为向量的乘积形式。此过程在之后讲述。

## SH functions 的性质





