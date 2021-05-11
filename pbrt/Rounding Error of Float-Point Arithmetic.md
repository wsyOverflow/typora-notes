## 浮点数运算带来的误差分析及其处理

### 浮点误差介绍

由于浮点数精度有限，无法正确表示全部实数集，在进行浮点数运算时，经常会出现舍入操作，这就导致了运算结果与实际结果有一定的误差。这种误差会随着浮点运算而累积，在某些场景中可能会导致严重错误。以光线求交操作为例：

- 如果一个计算来的交点的 $t$ 值虽然准确值为正，但由于浮点数运算引入的误差变为了负数。那么该交点就被错误地忽略丢弃。

- 计算得到的 ray-shape 交点可能会偏离实际表面，而位于其上侧或下侧。在交点处生成新光线(shadow/reflection ray)时，如果交点由于误差而位于表面下侧，就会导致新光线与表面重复相交。如果交点由于误差而在表面上侧偏离过远，shadow 和 reflection 会出现 detached 现象。如下图所示

  <img src="Rounding Error of Float-Point Arithmetic.assets/image-20210510204159979.png" alt="image-20210510204159979" style="zoom:50%;" />

#### 简单的阈值法

解决上述问题的一个经典做法是对生成的光线平移一个固定的 "ray epsilon"，忽略任何比 $t_{min}$ 近的交点。$t_{min}$ 的大小与光线起点到重复交点距离有关，如下图所示，则需要一个较大的 $t_{min}$。

<img src="Rounding Error of Float-Point Arithmetic.assets/image-20210510204733686.png" alt="image-20210510204733686" style="zoom:50%;" />

但较大的 $t_{min}$ 可能会导致一个正确交点被忽略。

### 浮点运算误差分析

#### 1. IEEE754 浮点数(只考虑单精度)

<img src="Rounding Error of Float-point arithmetic.assets/image-20210510103605519.png" alt="image-20210510103605519" style="zoom: 67%;" />

如上图所示，单精度浮点数由23位尾数(significand) $m$、8位指数(移码) $E$ 和1位符号位 $s$ 组成，$s\times 1.m\times 2^{E-127}$，令 $e=E-127$。

由于浮点数精度有限，对于两个计算机中连续的浮点数，也就是尾数部分的比特相差 $1$，这两个连续浮点数具有间隔。例如浮点数 $1$ 和 $2$，$e=0$，那么这个间隔就是 $2^{-23}$，这个间隔也被称作 **magnitude of a unit in last space(ulp)**。当然 ulp 的量级与浮点数值相关，即 $2^{e-23}$。

**将浮点数提升/下降到下一个连续浮点数：**以32位浮点数为例，将浮点数按位内存拷贝到32位无符号整数，然后根据浮点数的符号选择加/减1，再按位拷贝回浮点数。

#### 2.浮点数运算

符号定义：使用 $\oplus,\ominus,\otimes,\oslash, sqrt$ 表示数学运算 $+,-,\times,/,\sqrt{\space \space \space}$ 对应的浮点数运算，有

​                              $$a\oplus b = round(a+b) \\ a\ominus b=round(a-b) \\ a\otimes b = round(a\times b) \\ a \oslash b = round(a/b) \\ sqrt(a) = round(\sqrt{a})$$

其中，$round(x)$ 表示将一个实数舍入到最近的浮点数。

我们可以用实数的区间来表示浮点数运算带来的误差范围，以浮点数加法为例，

$$a\oplus b = round(a+b)\subset (a+b)(1\pm \epsilon)=[(a+b)(1-\epsilon),(a+b)(1+\epsilon)]$$

浮点加法引入的误差不会超过 $a+b$ 的 floating-point 间隔的一半，即 ulp 的一半。对于 32 位浮点数，我们可以使用  ulp 定义 $a+b$ 的浮点数间隔： $(a+b)2^{-23}$ 。因此，一半间隔为 $(a+b)2^{-24}$，令 $\epsilon_m=2^{-24}$ ，有

​           $$a\oplus b = round(a+b)\subset (a+b)(1\pm \epsilon_m)=[(a+b)(1-\epsilon_m),(a+b)(1+\epsilon_m)]$$

> 这里的浮点数间隔与第1部分的ulp定义略有不同，但结果一样。例如区间下限为 $(a+b)-(a+b)\epsilon_m$，$(a+b)\epsilon_m$ 将 $(a+b)$ 的尾数右移24位(因为阶码要与(a+b)对齐才可运算)，正好将隐含的最高位移动到23位尾数的后移一位，即舍入位。误差大小看的就是舍入位。

浮点数运算的一些性质，

$$1\otimes x=x \\ x\oslash x = 1 \\ x\oplus 0 = x \\ x\ominus x = 0$$

$$2\otimes x,x\otimes 2$$ 是没有误差的，更一般地，任何乘法或除法的另一个操作数为2的次幂，最终运算结果都不会引入误差。

#### 3. Error Propagation(数值分析)

两种误差度量：absolute error 和 relative error。假设实数运算结果为 $a$，其对应的浮点数运算结果为 $\tilde{a}$，有定义

**绝对误差：**$\delta_a = |\tilde{a}-a|$。**相对误差：**$\delta_r=|\frac{\tilde{a}-a}{a}|=|\frac{\delta_a}{a}|$。$a\neq 0$

将浮点运算结果看作经过扰动的实数运算结果：$\tilde{a}=a\pm \delta_a=a(1\pm \delta_r)$。以运算 $r=(((a+b)+c)+d)$ 为例，

$$\begin{align*} (((a\oplus b)\oplus c)\oplus d)\subset ((((a+b)(1\pm \epsilon_m)+c)(1+\epsilon_m)+d)(1\pm \epsilon_m)) \\ =(a+b)(1\pm \epsilon_m)^3+c(1\pm \epsilon_m)^2+d(1\pm \epsilon_m)  \\ \subset (a+b)(1\pm 4\epsilon_m)+c(1\pm 3\epsilon_m)+d(1\pm 2\epsilon_m) \\ = a+b+c+d+[\pm 4\epsilon_m (a+b)\pm 3\epsilon_m c \pm 2\epsilon_m d] \end{align*}$$

> 因为 $\epsilon_m$ 非常小，因此  $\epsilon_m$ 的次幂可以放大为加法，有以下不等式
>
> $$(1\pm \epsilon_m)^n \leq (1\pm (n+1)\epsilon_m)$$

上式方括号项与绝对误差有关：$4\epsilon_m |a+b|+ 3\epsilon_m |c| + 2\epsilon_m |d|$

可以看出 $a+b$ 项贡献了误差的最大部分，由此可看出，浮点运算先计算数值小的部分可以减小误差。通常同级运算编译器会有优化。使用 $(1\pm (n+1)\epsilon_m)$ 来作为 $(1\pm \epsilon_m)^n$ 的界限得到的误差过于保守，即区间过于松弛。

**更加紧凑的方法**是 $(1\pm \epsilon_m)^n$ 限定为 $1+\theta_n$，其中 $|\theta_n|\leq \frac{n\epsilon_m}{1-n\epsilon_m}$，$n\epsilon_m<1$。

令 $\gamma_n=\frac{n\epsilon_m}{1-n\epsilon_m}$，上例运算得到的扰动区间为 $|a+b|\gamma_3+|c|\gamma_2+|d|\gamma_1$

并且这种方法可以得到除法 $\frac{(1\pm \epsilon_m)^m}{(1\pm \epsilon_n)^n}$  限定为 $(1\pm \gamma_{m+n})$

已知两个带有之前运算累积的误差的数值 $a(1\pm \gamma_i),b(1\pm \gamma_j)$，乘法运算有

$$a(1\pm \gamma_i)\otimes b(1\pm \gamma_j) \subset ab(1\pm r_{i+j+1})$$

> $(1\pm \gamma_i)(1\pm \gamma_j)\subset (1\pm r_{i+j+1})$

这个结果得到的相对误差 $|\frac{ab\gamma_{i+j+1}}{ab}|=\gamma_{i+j+1}$，最终错误大概为 $(i+j+1)/2$ 个 ulps(相对于乘积结果)，结果如我们对乘法结果期望(次幂)，较为紧凑。

但对于加/减法运算，相对误差可能会增大很多，对于 $a(1\pm \gamma_i)\oplus b(1\pm \gamma_j)$，如果 $a,b$ 符号相同(减法时符号相反)，决定误差可被bound 为 $|a+b|\gamma_{i+j+1}$，相对误差大概为计算结果的 $(i+j+1)/2$ 个 ulps。

但当 $a,b$ 符号相反(减法时符号相同)时，相对误差会非常高。考虑 $a \approx -b$，相对误差 $\frac{|a|\gamma_{i+1}+|b|\gamma_{j+1}}{a+b}\approx \frac{2|a|\gamma_{i+j+1}}{a+b}$ 

#### 4. Running Error Analysis

除了使用代数形式计算 error bound，还可以在进行浮点运算时计算。背后思想：每次执行浮点操作时，根据当前浮点数已经携带的绝对误差，计算浮点运算结果的绝对误差(误差累积)，进而得到运算结果的error interval。

已知参与浮点运算的两个浮点数$a,b$，以及浮点数 $a$ 携带的绝对误差为 $\delta_a$，浮点数 $b$ 携带的绝对误差为 $\delta_b$ 。假设浮点运算结果的绝对误差为 $err$，那么可以得到运算结果的 error interval 为 $[v-err,v+err]$。但 $v\pm err$ 运算依然会引入误差，为了得到 sound error interval，进行一个保守处理——将区间下限下调一个 ulp，将上限上调一个 ulp，即 [NextUlpDown(v-err), NextUlpUp(v+err)]​。虽然这种操作可能会导致区间松弛，但能保证其 soundness 属性。下面介绍不同浮点运算的绝对误差计算方法，

- 加法运算 

  $(a\pm \delta_a)\oplus (b\pm \delta_b)=a+b+[\pm\delta_a\pm\delta_b\pm(a+b)\gamma_1\pm\gamma_1\delta_a\pm\gamma_1\delta_b]$

  运算结果的绝对误差可被 bound 为 $err = \delta_a+\delta_b+\gamma_1(|a+b|+\delta_a+\delta_b)$

### 求交运算中的浮点数误差处理

#### 1. Conservative Ray-Bounds Intersections

光线与 bounding box 求交，计算得到 $t_{min},t_{max}$ 。如果 $t_{min}<t_{max}$ ，那么说明光线穿过了 bounding box。试想，如果由于求交过程中涉及的大量浮点数运算而累积的误差导致 $t_{min}>t_{max}$ ，那么本次求交操作将得到一个不正确的 false 返回结果。如果与 Bounding Box 的交点被错误地丢弃，那么其内部的 Primitives 都会被忽略，这会导致失去很多光线交点。因此需要采取一些保守的做法，避免这种情况发生。

回想光线与 Bounding Box 求交计算，对于垂直于 $x$ 轴的 slab，$t=\frac{x-O_x}{\mathbf{d}_x}$，进行浮点数运算误差分析有

​							$t=(x\ominus O_x)\otimes (1\oslash \mathbf{d}_x)\subset \frac{x-O_x}{\mathbf{d}_x}(1\pm \epsilon)^3$

因此该运算结果的 Error Interval 为 $t(1\pm \gamma_3)$

当 $t_{min},t_{max}$ 的 Error Interval 重叠时，则有可能出现上述错误判断结果，如下图所示

<img src="Rounding Error of Float-Point Arithmetic.assets/image-20210511111350166.png" alt="image-20210511111350166" style="zoom:50%;" />

为了不丢弃正确的求交结果，一种保守做法：比较时用 $t_{max}$ 的 Error Interval 的上限与 $t_{min}$ 的 Error Interval 的下限进行比较，也可直接将 $t_{max}$ 的 Error Bound 的上限提升为两个 $\gamma_3$，而 $t_{min}$ 不采取额外操作。即

​									$t_{max}(1+ 2*\gamma_3) > t_{min}$

#### 2. Robust Triangle Intersections

三角形与光线求交过程中，我们先将三角形变换到  Ray Coordinate，再判断 $(0,0)$ 是否在三角形内部。对于变换操作，由于是所有三角形都会被变换，且误差较小，可忽略不计。对于判断 $(0,0)$ 点是否在三角形内部，需要根据 edge function 的符号判断：$e_0(P)=	P_{0x}P_{1y}-P_{0y}P_{1x}$

如果 $P_{0x}P_{1y}>P_{0y}P_{1x}$，那么经过浮点数的 round，$P_{0x}P_{1y}$也一定大于或等于 $P_{0y}P_{1x}$，因为浮点数向距离最近的浮点数进行 round 操作。如果由于浮点数 round 而导致 $P_{0x}P_{1y}=P_{0y}P_{1x}$，即 $e_0(P)=0$，此时提高精度(float->double) 再次进行计算。

#### 3. Bounding  Intersection Point Error

