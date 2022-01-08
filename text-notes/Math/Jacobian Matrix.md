## Summary

雅可比矩阵(Jacobian Matrix) 是一阶偏导在高维的推广形式，即向量微积分中的一阶偏导的矩阵形式，其行列式成为雅可比矩阵。

## 基础定义

假设一个从 n 维欧氏空间映射到 m 维欧氏空间的函数，$\boldsymbol{F}:\mathbb{R}^n\rightarrow\mathbb{R}^m$。$\boldsymbol{F}$ 可表示为以下方程组，
$$
\begin{cases}
y_1=f_1(x_1,x_2,\cdots,x_n) \\
y_2=f_2(x_1,x_2,\cdots,x_n) \\
\cdots\\
y_m=f_m(x_1,x_2,\cdots,x_n) \\
\end{cases}
$$
假设这些函数的一阶偏导数都存在，则可以使用雅可比矩阵 $\boldsymbol{J}_{\boldsymbol{F}}(x_1,x_2,...,x_n)$ 表示，并且有
$$
\boldsymbol{J}_{\boldsymbol{F}}(\boldsymbol{x}) = \boldsymbol{J}_{\boldsymbol{F}}(x_1,x_2,\cdots,x_n)=\frac{\partial(y_1,y_2,\cdots,y_m)}{\partial(x_1,x_2,\cdots,x_m)}=
\begin{pmatrix}
\frac{\part y_1}{\part x_1} & \frac{\part y_1}{\part x_2} & \cdots & \frac{\part y_1}{\part x_n} \\
\frac{\part y_2}{\part x_2} & \frac{\part y_2}{\part x_2} & \cdots & \frac{\part y_2}{\part x_n} \\
\vdots & \vdots & \ddots & \vdots \\
\frac{\part y_m}{\part x_1} & \frac{\part y_m}{\part x_2} & ... & \frac{\part y_m}{\part x_n}
\end{pmatrix}
\tag{1} \label{Jacobian Matrix def}
$$

同样有微分定义，假设 $\boldsymbol{p}$ 点可微，那么有
$$
\lim_{\boldsymbol{x}\rightarrow\boldsymbol{p}} \boldsymbol{F}(\boldsymbol{x})=\boldsymbol{F}(\boldsymbol{p})+\boldsymbol{J}_{\boldsymbol{F}}(\boldsymbol{p})\cdot(\boldsymbol{x}-\boldsymbol{p}) \tag{2} \label{differential}
$$
特殊地，当 $m=n$ 时 $\boldsymbol{J}_{\boldsymbol{F}}(\boldsymbol{p})$ 变成了一个方阵，对 $\eqref{differential}$ 式进行变换有
$$
\lim_{\boldsymbol{x}\rightarrow\boldsymbol{p}} \boldsymbol{F}(\boldsymbol{x})-\boldsymbol{F}(\boldsymbol{p})=\boldsymbol{J}_{\boldsymbol{F}}(\boldsymbol{p})\cdot(\boldsymbol{x}-\boldsymbol{p}) \\
d\boldsymbol{y}=\boldsymbol{J}_{\boldsymbol{F}}(\boldsymbol{p}) d\boldsymbol{x}
$$
其中，
$$
\begin{cases}
d\boldsymbol{y} = [dy_1,dy_2,\cdots,dy_n]^T \\
d\boldsymbol{x} = [dx_1,dx_2,\cdots,dx_n]^T
\end{cases}
$$
$\eqref{Jacobian Matrix def}$ 代入  $d\boldsymbol{y} = \boldsymbol{J}_{\boldsymbol{F}}(\boldsymbol{p}) d\boldsymbol{x}$ 中有，
$$
\begin{align}
\begin{pmatrix} dy_1 \\ dy_2 \\ \vdots \\ dy_n\end{pmatrix} &= 
\begin{pmatrix}
\frac{\part y_1}{\part x_1} & \frac{\part y_1}{\part x_2} & \cdots & \frac{\part y_1}{\part x_n} \\
\frac{\part y_2}{\part x_2} & \frac{\part y_2}{\part x_2} & \cdots & \frac{\part y_2}{\part x_n} \\
\vdots & \vdots & \ddots & \vdots \\
\frac{\part y_n}{\part x_1} & \frac{\part y_n}{\part x_2} & ... & \frac{\part y_n}{\part x_n}
\end{pmatrix}
\begin{pmatrix}dx_1 \\ dx_2 \\ \vdots \\ dx_n \end{pmatrix} \\
&=
\begin{pmatrix}
\frac{\part y_1}{\part x_1}dx_1 + \frac{\part y_1}{\part x_2}d_2 + \cdots + \frac{\part y_1}{\part x_n}dx_n \\
\frac{\part y_2}{\part x_1}dx_1 + \frac{\part y_2}{\part x_2}d_2 + \cdots + \frac{\part y_2}{\part x_n}dx_n \\
\vdots \\
\frac{\part y_n}{\part x_1}dx_1 + \frac{\part y_n}{\part x_2}d_2 + \cdots + \frac{\part y_n}{\part x_n}dx_n
\end{pmatrix}
\end{align}
$$
将上述向量写成基于正交的单位向量的形式：
$$
\begin{pmatrix}
dy_1 & 0 & \cdots & 0 \\
0 & dy_2 & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & dy_n
\end{pmatrix}
=
\begin{pmatrix}
\frac{\part y_1}{\part x_1}dx_1 & \frac{\part y_1}{\part x_2}dx_2 & \cdots & \frac{\part y_1}{\part x_n}dx_n \\
\frac{\part y_2}{\part x_1}dx_1 & \frac{\part y_2}{\part x_2}dx_2 & \cdots & \frac{\part y_2}{\part x_n}dx_n \\
\vdots & \vdots & \ddots & \vdots\\
\frac{\part y_n}{\part x_1}dx_1 & \frac{\part y_n}{\part x_2}dx_2 & \cdots & \frac{\part y_n}{\part x_n}dx_n
\end{pmatrix}
$$
左右两侧取行列式，左侧为微元的体积，为正值，右侧需要加上绝对值，有
$$
\begin{vmatrix}
dy_1 & 0 & \cdots & 0 \\
0 & dy_2 & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & dy_n
\end{vmatrix}
=
\begin{Vmatrix}
\frac{\part y_1}{\part x_1}dx_1 & \frac{\part y_1}{\part x_2}dx_2 & \cdots & \frac{\part y_1}{\part x_n}dx_n \\
\frac{\part y_2}{\part x_1}dx_1 & \frac{\part y_2}{\part x_2}dx_2 & \cdots & \frac{\part y_2}{\part x_n}dx_n \\
\vdots & \vdots & \ddots & \vdots\\
\frac{\part y_n}{\part x_1}dx_1 & \frac{\part y_n}{\part x_2}dx_2 & \cdots & \frac{\part y_n}{\part x_n}dx_n
\end{Vmatrix}
$$

$$
\begin{vmatrix}
dy_1 & 0 & \cdots & 0 \\
0 & dy_2 & \cdots & 0 \\
\vdots & \vdots & \ddots & \vdots \\
0 & 0 & \cdots & dy_n
\end{vmatrix}
=
\begin{Vmatrix}
\frac{\part y_1}{\part x_1} & \frac{\part y_1}{\part x_2} & \cdots & \frac{\part y_1}{\part x_n} \\
\frac{\part y_2}{\part x_1} & \frac{\part y_2}{\part x_2} & \cdots & \frac{\part y_2}{\part x_n} \\
\vdots & \vdots & \ddots & \vdots\\
\frac{\part y_n}{\part x_1} & \frac{\part y_n}{\part x_2} & \cdots & \frac{\part y_n}{\part x_n}
\end{Vmatrix}dx_1dx_2\cdots dx_n
$$

微元体积：
$$
dy_1dy_2\cdots dy_n=|\boldsymbol{J}_{\boldsymbol{F}}(\boldsymbol{x})|dx_1dx_2\cdots dx_n
$$
由上式可知，**在积分中可以使用雅可比矩阵进行积分换元**。
