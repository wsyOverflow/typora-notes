## Summary

## Photometry

在说明 Photometry(光度测量) 之前先来回顾以下辐射度量。Radiometry(辐射度量) 是对电磁辐射能量、功率的度量，下面表格是部分相关物理量，详细描述查看 [[1]](#[1])

| 物理量                              | 公式                                                         | 单位                           |
| ----------------------------------- | ------------------------------------------------------------ | ------------------------------ |
| Radiant Flux(辐射通量)或Power(功率) | $\Phi=\frac{dQ}{dt}$                                         | $W$(瓦特) 或 lm                |
| Irradiance(辐照度)                  | $E(x)=\frac{d\Phi (x)}{dA}$                                  | $W/m^2$ 或 lux(照度)           |
| Radiant Intensity(辐射强度)         | $I=\frac{\Phi}{4\pi}$                                        | $W/sr$ 或 cd(烛光)             |
| Radiance(辐射率)或luminance(亮度)   | $L(p,\omega)=\frac{d^2\Phi (p,\omega)}{d\omega dA cos\theta}$ | $W/(m^2\cdot sr)$ 或 nit(尼特) |

辐射度量并未考虑到人眼感知，而 Photometry 专门研究光强弱，将辐射量转换为光度测量单位，有如下对应关系：

| 辐射量：单位                        | 光量：单位                              |
| ----------------------------------- | --------------------------------------- |
| Radiant Flux(辐射通量)：$W$(瓦特)   | Luminous flux(光通量)：lumen(lm)        |
| Irradiance(辐照度)：$W/m^2$         | Illuminance(照度)：lux                  |
| Radiant Intensity(辐射强度)：$W/sr$ | Luminous Intensity(光强度)：candela(cd) |
| Radiance(辐射率)：$W/(m^2\cdot sr)$ | Luminance(亮度)：$cd/m^2$(nit)          |

其中通常使用 Luminance 来表示一个平面的亮度。例如 HDR 电视屏幕的峰值亮度范围为 500~1000 nit，天空亮度大约为 8000 nit，一个60瓦灯泡的亮度为 120000 nit，太阳光在地平线的亮度大约为 600000 nit。

**一般在图形学中不对辐射量和光量进行区分**。

## Spectral Power Distribution(SPD)

对于大多数光波都包含了不同的波长，通常以 SPD(光谱功率) 来表示光。光谱功率表示不同波长下的能量分布，如下图所示的可见光波长范围的能量分布：

<img src="Color And Radiometry.assets/image-20211231161802593.png" alt="image-20211231161802593" style="zoom:67%;" />

<center>顶部表示绿色单波长的能量；中部和下部都表示标准 D65 白光的能量分布，下部在可见光全波段都有能量分布，而中部只在红绿蓝三个波长有能量分布</center>

由上图可知，一种颜色可能对应多种光谱功率。**在图形学中，不会直接使用光谱功率表示颜色，而是使用 RGB。**

## SPD to RGB

人眼视网膜有三种不同的锥受体，每一种锥受体只处理一种波长的光，因此我们只需要 RGB 三个变量即可描述人眼能感知的所有颜色。通过修改对 RGB 三种波长的能量即可得到不同的颜色，CIE 组织进行色彩匹配实验，得到不同视觉颜色的光波在 RGB 三种波长上的能量分布。

### 1.色彩匹配实验

<img src="Color And Radiometry.assets/image-20211231165224253.png" alt="image-20211231165224253" style="zoom:50%;" />

上图是实验仪器示意图，上方三种波长的能量发射器，下发发出待匹配的视觉颜色，测试人员通过视角为 2° 的孔观察左侧白板，调控红绿蓝光的能量直至观察不出与待测色的区别。测试人员也可以选择在黑挡片上方投影颜色光，表示增加某波长的能量；也可以在下方投影颜色光，表示减少某波长的能量













## Reference

<a name="[1]">[1]</a> [Basic Radiometry](./Basic Radiometry.md)

