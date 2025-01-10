+++
title = "现代量子力学笔记(六): 时间演化和Shrödinger方程"
Tags = ["note", "physics", "quantum"]
Categories = ["note"]
date = 2018-12-26
+++
该部分笔记与Sakurai现代量子力学(2.1)对应

## 时间演化算符
首先要明确的是,时间在量子力学中是作为一个参数(parameter)出现的,而不是一个算符(operator).
因此,时间不是一个观测量(observable).

我们用时间演化算符`$\mathcal{U}(t,t_0)$`来描述状态随时间的变化.
`$$|α,t_0;t⟩=\mathcal{U}|α,t_0⟩.$$`

由于一个状态在不同时间都要满足归一化,所以`$\mathcal{U}$`是幺正的(unitary).
`$$\mathcal{U}(t,t_0)^†\mathcal{U}(t,t_0)=1$$`

根据时间的特性,时间演化算符应该是可连续结合的(composition property):
`$$\mathcal{U}(t_2,t_0)=\mathcal{U}(t_2,t_1)\mathcal{U}(t_1,t_0),\quad (t_2>t_1>t_0).$$`

考虑无穷小的时间演化:
`$$\lim_{dt→0}\mathcal{U}(t_0+dt,t_0)=1.$$`

根据以上关系,可以找到下面关系满足时间演化算符的上述特性:
`$$\mathcal{U}(t_0+dt,t_0)=1-iΩdt,$$`
其中`$Ω$`是厄米的:
`$$Ω^†=Ω.$$`

那么`$Ω$`有什么物理意义呢?首先,它有频率量纲,其次对比经典力学中Hamiltonian是时间演化的起因,
通过`$E=ħω$`可以认为:
`$$Ω=\frac{H}{ħ}$$`
因此:
`$$\mathcal{U}(t_0+dt,t_0)=1-\frac{iHdt}{ħ}.$$`

## Shrödinger方程
根据上式可写微分方程:
`$$\mathcal{U}(t+dt,t_0)=\mathcal{U}(t+dt,t)\mathcal{U}(t,t_0)=\left(1-\frac{iHdt}{ħ} \right)\mathcal{U}(t,t_0),$$`
可得:
`$$iħ\frac{∂}{∂t}\mathcal{U}(t,t_0)=H\mathcal{U}(t,t_0).$$`
这就是时间算符对应的Schödinger方程.

将算符作用在态`$|α,t_0⟩$`上:
`$$iħ\frac{∂}{∂t}|α,t_0;t⟩=H|α,t_0;t⟩.$$`
是体系状态的Schrödinger方程.

只要得到求解得到时间演化算符的具体表达,则只要知道初态,就能用时间演化算符得到后续各态.
微分方程的求解有下列三种情况:

- Hamiltonian算符与时间无关
- Hamiltonian算符与时间有关但各个时间下的算符对易
- Hamiltonian算符与时间有关且各个时间下的算符不对易

第一种情况对应的解的形式为:
`$$\mathcal{U}(t,t_0)=\exp\left[\frac{-iH(t-t_0)}{ħ} \right].$$`

第二种情况对应的解的形式为:
`$$\mathcal{U}(t,t_0)=\exp\left[-\frac{-i}{ħ}∫_{t_0}^t dt'H(t') \right].$$`

第三种情况对应的解的形式为:
`$$\mathcal{U}(t,t_0)=1+\sum_{n=1}^∞ \left(\frac{-i}{ħ}\right)^n ∫_{t_0}^t dt_1∫_{t_0}^{t_1} dt_2\cdots∫_{t_0}^{t_{n-1}} dt_n H(t_1)H(t_2)\cdots H(t_n). $$`
又叫做 *Dyson series*

## 能量本征值
有算符A与H对易,`$[A,H]=0$`,则:
`$$|α,t_0=0;t⟩=\sum_{a'}|a'⟩⟨a'|α⟩\exp\left(\frac{-iE_{a'}t}{ħ} \right).$$`

上式说明随着时间的流逝,一个体系的状态总是可以写成原始的本征态的叠加,叠加的系数只是一个相位上的改变.

## 期望随时间演化
现在考虑某算符B的期望值`$⟨B⟩$`随时间的演化情况,分为两种可能:

1. 初始态是某个算符A的本征态(定态(stationary state))
2. 初始态是某个算符A的本征态的叠加(非定态(nonstationary state))

在这里,B和A无需是对易的.

对于情况(1):
`$$⟨B⟩=⟨a'|B|a'⟩,$$`

对于情况(2):
`$$⟨B⟩=\sum_{a'}\sum_{a''}c^{*}_{a'}c_{a''}⟨a'|B|a''⟩\exp{} \left(\frac{-i(E_{a''}-E_{a'})t}{ħ} \right)$$`
`$$ω_{a''a'}=\frac{(E_{a''}-E_{a'})}{ħ}.$$`

## 自旋进动
设有体系在算符`$S_z$`的作用下随时间演化.
初始态为:`$S_z$`本征态的叠加:
`$$|ψ_0⟩ = |0⟩ + |1⟩ = |↓⟩ + |↑⟩.$$`
则体系对于算符`$S_x,S_y$`的期望分别如下所示.
![](https://raw.githubusercontent.com/unkcpz/images/master/zpcknu-blog/spinprecession.png)
[spin-precession](https://github.com/unkcpz/images/blob/master/zpcknu-blog/spin_procession.ipynb)

## Correlation Amplitude and the Energy-Time Uncertainty Relation
