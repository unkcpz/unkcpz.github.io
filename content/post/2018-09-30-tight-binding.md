---
title: "Tight binding 紧束缚模型及其实例"
date: '2018-09-30'
categories:
    - post
tags:
    - physics 
    - condensed-matter
---

## 紧束缚模型

[紧束缚模型(Tight binding)](https://en.wikipedia.org/wiki/Tight_binding)是计算周期性体系电子结构的一种方法。 它和[原子轨道线性组合方法 (LCAO)](https://en.wikipedia.org/wiki/Linear_combination_of_atomic_orbitals)
方法相似。即便这种近似方法忽略了电子-电子相互作用，它还是能够达到特定精度的结果，并且可作为其他
更为精确方法的出发点。
同时，紧束缚可能是最早的描述电子结构的模型，它在1928年被Bloch提出并使用。也因此，紧束缚模型和Bloch定理
紧密相关。

经验的紧束缚模型给出了描述体系的矩阵形式，矩阵的参数可以通过实验数据或者其他理论方法得到。在模型中，
除了体系轨道的对称性，无需其余信息。

$$
\psi_{ucell}(\mathbf{r}) = \sum_{a} \sum_{ao} c_{ao,a} \, \phi_{ao}^{Z_a}(\mathbf{r}-\mathbf{r_a})
$$

紧束缚的“紧”描述的是体系中的电子是和该电子所在的原子紧紧束缚在一起的。因此，该电子与相邻原子只有
有限的作用。

原胞中的波函数 $\psi_{ucell}(\mathbf{r})$ 可以认为由晶胞中所有原子价轨道线性组合而成:

$$\psi_{ucell}(\mathbf{r})=\displaystyle\sum_{a} \sum_{ao}c_{ao,a}\phi_{ao}^{Z_a}(\mathbf{r}-\mathbf{r_a})$$

其中，$a$为对所有原子求和，$ao$为对所有原子轨道求和。上式可以被简写为对所有轨道求和:

$$\psi_{ucell}(\mathbf{r})=\displaystyle\sum_i c_i \phi_i (\mathbf{r}-\mathbf{r_i})$$

其中参数 $c_i$ 可通过将波函数代入Schrödinger方程获得。比如，以 $\mathrm{CaSO}_4$ 为例，
其价轨道分别为Ca原子的4s轨道，硫原子的3s轨道和3个3p轨道，氧原子的2s轨道和3个2p轨道。因此，
对于一份 $\mathrm{CaSO}_4$ 共有 $1+4+4\times 4=21$ 项波函数参数项，($i=1,\dots, 21$)。

使用[Bloch定理](https://en.wikipedia.org/wiki/Bloch_wave)，波函数在整个空间中可以写为：

$$
\psi_{\mathbf{k}}(\mathbf{r})=\frac{1}{\sqrt{N}}\displaystyle\sum_{h,j,l} e^{i(h\mathbf{k}\cdot \mathbf{a_1} + j\mathbf{k}\cdot \mathbf{a_2} + l\mathbf{k}\cdot \mathbf{a_3})} \psi_{ucell} (\mathbf{r}-h\mathbf{a_1}-j\mathbf{a_2}-l\mathbf{a_3}).
$$

其中 $N$ 为实际晶体中原胞的数量（因为全空间为无限大的胞来表示周期性晶体，在全空间积分后则其被积分掉）
；$h,j,\mathrm{and}, l$ 用于表示晶体
中的特定原胞；$\mathbf{k}$ 为波矢；$\mathbf{a_1}, \mathbf{a_2}, \mathrm{and}, \mathbf{a_3}$ 为
实空间中原胞的基矢。晶体在空间中有平移对称性，因此平移算符和能量算符是对易的:

$$ 
\hat{T}_{pqs} \psi_{\vec{k}}\left(\vec{r}\right)=\psi_{\vec{k}}\left(\vec{r}+p\vec{a}_1+q\vec{a}_2+s\vec{a}_3\right)=e^{i\left(p\vec{k}\cdot\vec{a}_1 + q\vec{k}\cdot\vec{a}_2 + s\vec{k}\cdot\vec{a}_3\right)}\psi_{\vec{k}}\left(\vec{r}\right).
$$

紧束缚模型的波函数$\psi_{\vec{k}}$可代入不含时的Schrödinger求解能量和波矢的色散关系：
$$ \hat{H}\psi_{\vec{k}}=E\psi_{\vec{k}} .
$$

通过分别在左边乘上每个原子轨道的波函数$\phi_{n}^*\left(\vec{r}\right)$，可以构成方程组：
$$ \langle\phi_{n}|\hat{H}|\psi_{\vec{k}}\rangle = E\langle\phi_{n}|\psi_{\vec{k}}\rangle.
$$
其中的$n$为轨道的数量。

由于随着电子间距离的增大，轨道的交叠迅速减小。因此，可以简单的在模型中认为只有on-site项($\langle\phi_n|\hat{H}|\phi_n\rangle$)和最近邻项重要。故等式的左右两边可以化简为只有相应高阶项：
$$ c_n\langle\phi_{n}|\hat{H}|\phi_{n}\rangle + \sum\limits_{m=\text{nearest neighbors}}c_m\langle\phi_{n}|\hat{H}|\phi_{m}\rangle e^{i\left(h\vec{k}\cdot\vec{a}_1 + j\vec{k}\cdot\vec{a}_2 + l\vec{k}\cdot\vec{a}_3\right)} + \text{small terms} = Ec_n\langle\phi_{n}|\phi_{n}\rangle + \text{small terms} .
$$

对每一个原子轨道做如上操作，组成方程组，求解可得体系的能带。

#### 非正交项 (optional)

### 1-D 一维单原子晶体

考察拥有一个价层轨道 $\phi$ 构成的一维晶体。体系紧束缚的波函数可以写为：

$$\psi_{\mathbf{k}}(x)=\frac{1}{\sqrt{N}}\displaystyle\sum_{n}e^{inka}\phi (x-na)$$

此处，$n$ 为整数。将上述波函数带入Schrödinger方程，左右两边乘以 $\phi^{*}(x)$ 在全空间积分得：

$$\langle \phi(x) | \hat{H} | \psi_{\mathbf{k}}(x) \rangle= E \langle \phi(x) | \psi_{\mathbf{k}}(x) \rangle$$

在紧束缚近似中，只有on-site 和最近邻矩阵元在等号左边保留，只有on-site项在右边保留。其余项忽略，因此有：

$$\langle\phi\left(x\right)|\hat{H}|\phi\left(x-a\right)\rangle e^{-ika} +\langle\phi\left(x\right)|\hat{H}|\phi\left(x\right)\rangle+\langle\phi\left(x\right)|\hat{H}|\phi\left(x+a\right)\rangle e^{ika} + \text{small terms}= E + \text{small terms} .$$

令 $\epsilon = \langle\phi\left(x\right)|\hat{H}|\phi\left(x\right)\rangle$ 且 $t = - \langle\phi\left(x\right)|\hat{H}|\phi\left(x-a\right)\rangle$
色散关系可表示为

$$ E= \epsilon -t\left(e^{-ika} + e^{-ika}\right).
$$

也就是

$$ E= \epsilon -2t\cos\left(ka\right) .
$$

如下图($\epsilon=2, t=1, a=1$)：

![](/images/tb-1D-01.png)

### 1-D 一维双原子晶体

考察在一个一维晶胞中有两个原子体系。体系的波函数可以写为：

$$ \psi_{k}\left(x\right)=\frac{1}{\sqrt{N}}\sum\limits_{n}e^{inka} \left( c_1\phi_1\left(x-na\right) +c_2\phi_2\left(x-na\right)\right) .
$$

仿照一维单原子的处理方法，在Schrödinger方程的两边分别乘上原子波函数并在全空间积分：
$$ \begin{array}{a}
\langle\phi_1|\hat{H}|\psi_{k}\rangle = E\langle\phi_1|\psi_{k}\rangle , \\ \langle\phi_2|\hat{H}|\psi_{k}\rangle = E\langle\phi_2|\psi_{k}\rangle . \end{array}
$$

依旧只考虑on-site和最近邻作用，忽略其余项，可得：

$$ \begin{array}{a}
\epsilon_1 c_1 -tc_2(1+e^{-ika}) = Ec_1 ,\\
\epsilon_2 c_2 -tc_1(1+e^{ika}) = Ec_2.
\end{array} $$

其中，$\epsilon_1 = \langle\phi_1(x)|\hat{H}|\phi_1(x)\rangle$,$\epsilon_2 = \langle\phi_2(x)|\hat{H}|\phi_2(x)\rangle$，且$t = - \langle\phi_1(x)|\hat{H}|\phi_2(x)\rangle= - \langle\phi_2(x-a)|\hat{H}|\phi_1(x)\rangle$。需要注意，在此处的$t$相同的是因为假设该双原子分子的两个原子
相同，若不同则$t_1 \neq t_2$。上面的方程组可以写为矩阵的形式：
$$ \begin{bmatrix}
\epsilon_1 -E & -t\left(1+e^{-ika}\right) \\
 -t\left(1+e^{ika}\right) & \epsilon_2 -E\end{bmatrix}\left[ \begin{array}{c} c_1 \\ c_2 \end{array} \right] =0.$$
 行列式为零时有解，也就是:
 $$ E^2-(\epsilon_1+\epsilon_2)E + \epsilon_1\epsilon_2 -2t^2(1+\cos(ka))=0.
$$
可以解得色散关系为：
$$ E=\frac{(\epsilon_1+\epsilon_2)\pm \sqrt{(\epsilon_1-\epsilon_2)^2+8t^2(1+\cos(ka))}}{2}.
$$
如下图：

![](/images/1d-2atom.png)

### 2-D 石墨烯 (Graphene)
Graphene为二维六角晶格，其中原胞包含两个Carbon原子$C_1$和$C_2$，位于不等价的Wickoff位点。
在Graphene中，$p_x,p_y$轨道参与平面方向$sp^2$杂化，$p_z$轨道为价轨道。
两个原子的$p_z$轨道分别为$\phi_{2p_{z1}}$和$\phi_{2p_{z2}}$。

原胞的基矢为：
$$ \begin{array}{a}
\vec{a}_1 = \frac{\sqrt{3}a}{2} \hat{x}+ \frac{a}{2}\hat{y}, \\
\vec{a}_2 = \frac{\sqrt{3}a}{2} \hat{x}- \frac{a}{2}\hat{y}. \end{array}$$

因此，graphene紧束缚模型的波函数可以写为：
$$ \psi_{\vec{k}}\left(\vec{r}\right)=\frac{1}{\sqrt{N}}\sum\limits_{h,j}e^{i\left(h\vec{k}\cdot\vec{a}_1 + j\vec{k}\cdot\vec{a}_2\right)} \left( c_1\phi_{\text{2p}_{z1}}\left(\vec{r}-h\vec{a}_1-j\vec{a}_2\right) + c_2 \phi_{\text{2p}_{z2}}\left(\vec{r}-h\vec{a}_1-j\vec{a}_2\right) \right) .
$$

可代入不含时的Schrödinger，在全空间积分求解能量和波矢的色散关系.

$$ \begin{array}{a}
\langle\phi_{\text{2p}_{z1}}|\hat{H}|\psi_{k}\rangle = E\langle\phi_{\text{2p}_{z1}}|\psi_{k}\rangle , \\ \langle\phi_{\text{2p}_{z2}}|\hat{H}|\psi_{k}\rangle = E\langle\phi_{\text{2p}_{z2}}|\psi_{k}\rangle . \end{array}$$

依旧只考虑on-site和最近邻作用:

$$ \begin{array}{a}
\epsilon c_1 -tc_2\left(1+e^{-i\vec{k}\cdot\vec{a_1}} +  e^{-i\vec{k}\cdot\vec{a_2}}\right) = Ec_1 ,\\
\epsilon c_2 -tc_1\left(1+e^{i\vec{k}\cdot\vec{a_1}} +  e^{i\vec{k}\cdot\vec{a_2}}\right) = Ec_2.
\end{array} $$

其中$\epsilon = \langle\phi_{\text{2p}_{z1}}\left(\vec{r}\right)|\hat{H}|\phi_{\text{2p}_{z1}}\left(\vec{r}\right)\rangle$ 且 $t = - \langle\phi_{\text{2p}_{z1}}\left(\vec{r}\right)|\hat{H}|\phi_{\text{2p}_{z2}}\left(\vec{r}\right)\rangle= - \langle\phi_{\text{2p}_{z1}}\left(\vec{r}\right)|\hat{H}|\phi_{\text{2p}_{z1}}\left(\vec{r}-\frac{a}{\sqrt{3}}\hat{x}\right)\rangle$

同一维双原子体系，原胞内两个原子相同，所以其近邻关系相同，根据对称性，$t$在上式中相同。
使用矩阵表示上述方程组：
$$ \begin{bmatrix}
\epsilon -E & -t\left(1+e^{-i\sqrt{3}k_xa/2}e^{-ik_ya/2} +  e^{-i\sqrt{3}k_xa/2}e^{+ik_ya/2}\right) \\
 -t\left(1+e^{i\sqrt{3}k_xa/2}e^{ik_ya/2} +  e^{i\sqrt{3}k_xa/2}e^{-ik_ya/2}\right) & \epsilon -E\end{bmatrix}\left[ \begin{array}{c} c_1 \\ c_2 \end{array} \right] =0.
$$

行列式为零求解，使用$2\cos x = e^{ix} + e^{-ix}$，可得：

$$ \left(\epsilon-E\right)^2 -t^2\left(3+2\cos\left(\frac{\sqrt{3}k_xa}{2}-\frac{k_ya}{2}\right)+2\cos\left(\frac{\sqrt{3}k_xa}{2}+\frac{k_ya}{2}\right)+2\cos\left(k_ya\right)\right)=0.
$$

使用下面三角函数关系化简：

$$ \begin{array}{a}
\cos\left(a+b\right)=\cos\left(a\right)\cos\left(a+b\right)-\sin\left(a\right)\sin\left(a\right), \\\cos\left(a-b\right)=\cos\left(a\right)\cos\left(a+b\right)+\sin\left(a\right)\sin\left(a\right)\text{, and} \\
\cos\left(2a\right)=2\cos\left(a\right)-1,
\end{array} $$

可以得到最后的色散关系为：

$$E=\epsilon \pm t\sqrt{1+4\cos\left(\frac{\sqrt{3}k_xa}{2}\right)\cos\left(\frac{k_ya}{2}\right)+4\cos^2\left(\frac{k_ya}{2}\right)}.
$$

取高对称点$\Gamma, M, K$ 做图如下($\epsilon=0, t=1, a=1$)：

![](/images/graphene.png)

#### h-BN (optional)
可参照graphene的模型来给出h-BN的紧束缚解。

只改变on-site的参数($\epsilon_0=-1, \epsilon_1=1, t=1, a=1$)。可以得到如下的能带结构：

![](/images/h-XY.png)

模型的信息为：

```text
---------------------------------------
report of tight-binding model
---------------------------------------
k-space dimension           = 2
r-space dimension           = 2
number of spin components   = 1
periodic directions         = [0, 1]
number of orbitals          = 2
number of electronic states = 2
lattice vectors:
 #  0  ===>  [     1.0 ,     0.0 ]
 #  1  ===>  [     0.5 ,   0.866 ]
positions of orbitals:
 #  0  ===>  [  0.3333 ,  0.3333 ]
 #  1  ===>  [  0.6667 ,  0.6667 ]
site energies:
 #  0  ===>       1.0
 #  1  ===>      -1.0
hoppings:
<  0 | H |  1 + [  0 ,  0 ] >     ===>     -1.0 +     0.0 i
<  1 | H |  0 + [  1 ,  0 ] >     ===>     -1.0 +     0.0 i
<  1 | H |  0 + [  0 ,  1 ] >     ===>     -1.0 +     0.0 i
hopping distances:
|  pos(  0 )  - pos(  1 + [  0 ,  0 ] ) |  =    0.5774
|  pos(  1 )  - pos(  0 + [  1 ,  0 ] ) |  =    0.5774
|  pos(  1 )  - pos(  0 + [  0 ,  1 ] ) |  =    0.5774
```

### 3-D 锂 (bcc) 晶体
金属锂正常以体心立方(Body Centered Cubic --- BCC)密堆排列，
其原胞包含一个原子，原胞的基矢如下：

$$ \begin{array}{a}
\vec{a}_1 = \frac{a}{2} \left( \hat{x}+\hat{y}-\hat{z} \right), \\
\vec{a}_2 = \frac{a}{2} \left( -\hat{x}+\hat{y}+\hat{z} \right), \\
\vec{a}_3 = \frac{a}{2} \left( \hat{x}-\hat{y}+\hat{z} \right). \end{array}$$

锂元素在第二周期，价层仅有一个$2s$轨道，设为$\phi_{2s}$。其紧束缚波函数可以写为：

$$ \psi_{\vec{k}}\left(\vec{r}\right)=\frac{1}{\sqrt{N}}\sum\limits_{h,j,l}e^{i\left(h\vec{k}\cdot\vec{a}_1 + j\vec{k}\cdot\vec{a}_2 + l\vec{k}\cdot\vec{a}_3\right)} \phi_{\text{2s}}\left(\vec{r}-h\vec{a}_1-j\vec{a}_2-l\vec{a}_3\right).
$$

带入不含时Schrödinger方程并忽略紧邻以外的作用项可以得到：

$$ \epsilon - t e^{i\vec{k}\cdot\vec{a_1}} - t e^{i\vec{k}\cdot\vec{a_2}} - t e^{i\vec{k}\cdot\vec{a_3}} - t e^{-i\vec{k}\cdot\vec{a_1}} - t e^{-i\vec{k}\cdot\vec{a_2}} - t e^{-i\vec{k}\cdot\vec{a_3}} - t e^{i\vec{k}\cdot\left(\vec{a_1} +\vec{a}_2 +\vec{a}_3\right)} - t e^{-i\vec{k}\cdot\left(\vec{a_1} +\vec{a}_2 +\vec{a}_3\right)} + \text{small terms}= E + \text{small terms}.
$$

其中$\epsilon = \langle\phi_{\text{2s}}\left(\vec{r}\right)|\hat{H}|\phi_{\text{2s}}\left(\vec{r}\right)\rangle$, 且
$t = - \langle\phi_{\text{2s}}\left(\vec{r}\right)|\hat{H}|\phi_{\text{2s}}\left(\vec{r}-\vec{a}_1\right)\rangle$

上式的$t$由于对称性对全部近邻都相等。使用基矢的关系，上式可以化为：

$$ \begin{array}{b}
 E = \epsilon - te^{ik_xa/2}e^{ik_ya/2}e^{-ik_za/2} -te^{-ik_xa/2}e^{ik_ya/2}e^{ik_za/2} -te^{ik_xa/2}e^{-ik_ya/2}e^{ik_za/2}
-te^{-ik_xa/2}e^{-ik_ya/2}e^{ik_za/2} \\
-te^{ik_xa/2}e^{-ik_ya/2}e^{-ik_za/2} -te^{-ik_xa/2}e^{ik_ya/2}e^{-ik_za/2}
 -te^{ik_xa/2}e^{ik_ya/2}e^{ik_za/2} -te^{-ik_xa/2}e^{-ik_ya/2}e^{-ik_za/2}
\end{array} .$$

进一步利用$2\cos x = e^{ix} + e^{-ix}$化简为:

$$ E = \epsilon - 8t\cos\left(k_xa/2\right)\cos\left(k_ya/2\right)\cos\left(k_za/2\right) .
$$

选取高对称点$P, \Gamma, N, H, \Gamma$可以得到下左图的能带色散曲线：

与第一性原理的结果（下右图）的结果比较可以看出($\epsilon=0, t_0=-1, t_1=-0.1, a=1$:，

下图的`jupyter notebook`源码，请访问[BCC-Tight-Binding jupyter](https://github.com/unkcpz/images/blob/master/zpcknu-blog/bcc-tb.ipynb)

![](/images/bcc-tb.png)
