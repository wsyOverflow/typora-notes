#                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       辐射度量学基础 

Whitted style 光线追踪使用 Blinn-Phong 着色模型，着色效果不真实。因此有提出基于辐射度量学的着色模型，以物理正确的方式进行光照计算。

## 相关术语

| 物理量                              | 公式                                                         | 单位                           |
| ----------------------------------- | ------------------------------------------------------------ | ------------------------------ |
| Radiant Energy(辐射能)              | $Q:$电磁辐射能量                                             | $J$(焦耳)                      |
| Radiant Flux(辐射通量)或Power(功率) | $\Phi=\frac{dQ}{dt}$                                         | $W$(瓦特) 或 lm                |
| Angle(角度)                         | $\theta=\frac{l}{r}$                                         | rad(弧度)                      |
| Solid Angle(立体角)                 | $\Omega=\frac{A}{r^2}$                                       | sr(球面角度)                   |
| Radiant Intensity(辐射强度)         | $I=\frac{\Phi}{4\pi}$                                        | $W/sr$ 或 cd(烛光)             |
| Irradiance(辐照度)                  | $E(x)=\frac{d\Phi (x)}{dA}$                                  | $W/m^2$ 或 lux(照度)           |
| Radiance(辐射率)或luminance(亮度)   | $L(p,\omega)=\frac{d^2\Phi (p,\omega)}{d\omega dA cos\theta}$ | $W/(m^2\cdot sr)$ 或 nit(尼特) |

#### 1. Radiant Energy(辐射能量)

电磁辐射的能量，单位焦耳 $J$

#### 2. Radiant Flux/Power(辐射通量/功率)

单位时间内释放(emitted)、反射(reflected)、传播(transmitted)或接收(received)的能量，即功率

$$
\Phi = \frac{dQ}{dt}
$$

#### 3. Angle

圆上的弧长与半径的比值: $\theta=\frac{l}{r}$ ，圆有 $2\pi$ 弧度

<img src=".\Basic Radiometry.assets\image-20210418105514511.png" alt="image-20210418105514511" style="zoom:33%;" />

#### 4. Solid Angle(立体角)

球面上的投影面积与半径的平方之比: $\Omega = \frac{A}{r^2}$，球的立体角为 $4\pi$ 球面角度(steradians)

<img src=".\Basic Radiometry.assets\image-20210418105703119.png" alt="image-20210418105703119" style="zoom:33%;" />

#### 5. Differential Solid Angle(微分立体角)

<img src=".\Basic Radiometry.assets\image-20210418105937707.png" alt="image-20210418105937707" style="zoom: 25%;" />

- 单位面积：$dA=(rd\theta)(rsin\theta d\phi)=r^2sin\theta d\theta d\phi$，单位立体角对应的球面上单位区域的面积

- 单位立体角：$d\omega = \frac{dA}{A^2}=sin\theta d\theta d\phi$

- 球面的微分立体角：$\Omega=\int_{S^2}d\omega=\int_0^{2\pi}\int_0^{\pi}sin\theta d\theta d\phi=4\pi$，其中 $S^2$ 是求面积

  > $dA$ 的证明，$dA$ 可看作 $d\theta$ 和 $d\phi$ 对应的微分弧组成的小矩形，如下图中红色弧线与蓝色弧线
  >
  > <img src="Basic Radiometry.assets\2.PNG" alt="2" style="zoom:25%;" />
  >
  > 其中蓝色弧线位于半径为 $r_\phi$ 的小圆上，而红色弧线位于半径为 $r_\theta$ 的大圆上，又知道 $sin\theta = \frac{r_\phi}{r_\theta}$，由弧长公式有
  >
  > 红色弧：$r_\theta d\theta$，蓝色弧：$r_\phi d\phi=r_\theta sin\theta d\phi$。
  >
  > 因此 $dA=(r_\theta d\theta)(r_\theta sin\theta d\phi)=r_\theta^2sin\theta d\theta d\phi$。
  >
  > > $sin\theta$ 的直观理解是，越靠近极点位置，$r_\phi$ 越小，因此微分面积也越小；越靠近赤道位置，$r_\phi$ 越大，因此微分面积也越大

$\omega$ 作为单位立体角的方向向量

<img src=".\Basic Radiometry.assets\image-20210418111221992.png" alt="image-20210418111221992" style="zoom:25%;" />

- Isotropic Point Source(各向同性光源)：球面上各单位立体角辐射强度 (Radiant Intensity) 相同。整个球面的辐射通量/功率：$\Phi=\int_{S^2}Id\omega=4\pi I$，辐射强度 $I=\frac{\phi}{4\pi}$。

<img src=".\Basic Radiometry.assets\image-20210418111805822.png" alt="image-20210418111805822" style="zoom:25%;" />

#### 6. Radiant Intensity(辐射强度)

点光源**每立体角**发出的功率：$I(\omega)=\frac{d\Phi}{d\omega}$

#### 7. Irradiance(辐照度)

辐照度是每(垂直/投影)**单位面积**入射到一个表面上一点的辐射通量(功率)，$E(x)=\frac{d\Phi(x)}{dA}$

单位时间内光子离开或进入单位面积的通量

Lambert 余弦定律：表面辐照度与光方向和表面法线夹角的余弦值成正比

<img src="Basic Radiometry.assets\image-20210418113052481.png" alt="image-20210418113052481" style="zoom:25%;" />

> 这里 irrandiance 是单位面积入射到一点的辐射通量，$cos\theta$ 调整的是方向 $l$ 上入射功率的贡献，将方向 $l$ 上的辐射通量投影到接收点的法线方向 $n$ 上。

Irradiance 衰减：$E=\frac{\Phi}{4\pi r^2}$ ，$\Phi$ 记录的是单位半径球面在单位时间内所接收的能量的功率。二维示意图如下

<img src="Basic Radiometry.assets\image-20210418113447525.png" alt="image-20210418113447525" style="zoom:25%;" />



#### 8. Radiance(辐射率)

Radiance 用于描述光在环境中的分布的基本场量。辐射率(Radiance)或亮度(luminance) ：是指一个表面在**每单位立体角、每单位投影面积**上所发射(emitted)、反射(reflected)、透射(transmitted)或接收(received)的辐射通量(功率)。

$$L(p,\omega)=\frac{d^2\Phi (p,\omega)}{d\omega dA cos\theta}$$，Light traveling along a Ray                  <img src="Basic Radiometry.assets\image-20210418113916011.png" alt="image-20210418113916011" style="zoom:25%;" />

> 这里是从表面点 $p$ 沿着其某一单位立体角方向 $\omega$ 发出的功率，$cos\theta$ 是将辐射面投影到以 $\omega$ 为法线的平面。

- Incident Radiance(入射辐射)：到达表面的**单位立体角**的 irradiance(辐照度)，即 radiance。

​          $$L(p,\omega)=\frac{dE(p)}{d\omega cos\theta}$$                                                                    <img src=".\Basic Radiometry.assets\image-20210418140012081.png" alt="image-20210418140012081" style="zoom:25%;" />

> 沿着 $\omega$ 方向到达表面 $p$ 点的辐射，$cos\theta$ 将入射方向投影到表面点 $p$ 法线方向

- Exiting Radiance(出射辐射)：离开表面的**单位投影面积**的 Radiance Intensity(辐射强度)。如面光源

​		$$L(p,\omega)=\frac{dI(p,\omega)}{dAcos\theta}$$                                                                    <img src="Basic Radiometry.assets\image-20210418141032723.png" alt="image-20210418141032723" style="zoom:25%;" />

**注意**：Incident Radiance 与 Exiting Radiance 虽然表达式形式有所不同，但代入后最终都可转化为 $\frac{d^2\Phi (p,\omega)}{d\omega dA cos\theta}$。

> 具有单位立体角限制的 irradiance 等同于 radiance，具有单位投影面积的 Radiance Intensity 等同于 radiance

## 辐照度(Irradiance) VS. 辐射率(Radiance)

- Irradiance：在面积 $dA$ 的总辐射通量
- Radiance：在面积 $dA$ 、方向 $d\omega$ 上的辐射通量

​			$$dE(p,\omega)=L_i(p,\omega)cos\theta \space d\omega \\ E(p,\omega)=\int_{H^2}L_i(p,\omega)cos\theta \space d\omega$$                     <img src=".\Basic Radiometry.assets\image-20210418143001958.png" alt="image-20210418143001958" style="zoom:25%;" />

