+++
title = "现代量子力学笔记(三): 抽象到具体---真实的量子世界"
Tags = ["note", "physics", "quantum"]
Categories = ["note"]
date = 2018-12-21
+++
该部分笔记与Sakurai现代量子力学(1.4)对应

## 测量

Dirac 1985, p.36

    "A measurement always causes the system to jump into
    an eigenstate of the dynamical variable that is being
    measured."

在测量之前,体系被认为处于一个混合的状态`$|α⟩$`
`$$|α⟩=\sum_{a'}c_{a'}α⟩=\sum_{a'}|a'⟩⟨a'|α⟩$$`
测量使该混合态"塌缩"至任意一个可观测的态.

量子力学概率解释的一个公设为:我们不能在测量之前知道这个测量得到系统的具体状态,
我们只能知道系统跳到某一个状态的概率.
`$$\mathrm{Probability \ for \ a'=|⟨a'|α⟩|^2}$$`

对于状态`$|α⟩$`,其对与算符A的期望为`$⟨A⟩$`或`$⟨A⟩_α$`,写为:
`$$⟨A⟩≡⟨α|A|α⟩=\sum_{a'}\sum_{a''}⟨α|a''⟩⟨a''|A|a'⟩⟨a'|α⟩=\sum_{a'}a'|⟨a'|α⟩|^2$$`
上式最右边一项直观的可理解为测量直的加权平均.

## 对易非对易
定义:
`$$[A,B]≡AB-BA$$`
<!-- `$$\{A,B\}≡AB+BA$$` -->
若`$[A,B]=0,$`则称算符A,B对易,`$[A,B]\neq 0$`称算符A,B非对易.

### 对易(Compatible)
A,B对易,则矩阵元`$⟨a''|B|a'⟩$`为对角矩阵.:
`$$⟨a''|B|a'⟩=δ_{a'a''}⟨a'|B|a'⟩.$$`
算符B写为:
`$$B=\sum_{a''}|a''⟩⟨a''|B|a''⟩⟨a''|.$$`
B算符作用在A的其中一个本征向量`$|a'⟩$`上:
`$$B|a'⟩=\sum_{a''}|a''⟩⟨a''|B|a''⟩⟨a''|a'⟩=(⟨a'|B|a'⟩)|a'⟩$$`
说明A,和B有着相同的本征向量`$|a'⟩$`,
`$$b'=⟨a'|B|a'⟩$$`

A,B的测量不会互相破坏信息(do not interfere),如下:
`$$|α⟩\xrightarrow{\text{A measure}}|a',b'⟩\xrightarrow{\text{B measure}}|a',b'⟩\xrightarrow{\text{A measure}}|a',b'⟩.$$`

### 非对易(Incompatible)
若算符A,B非对易,则A,和B不会拥有相同的 *完备的* 本征向量,这里强调 *完备的*,是因为
非对易的A,B可能在子空间拥有相同的本征向量.

#### 另一个反直觉的实验


## 测不准关系(The Uncertainty Relation)
定义算符`$ΔA≡A-⟨A⟩$`,该算符的平方`$(ΔA)^2$`描述了算符A的dispersion:
`$$⟨(ΔA)^2⟩=⟨(A^2-2A⟨A⟩+⟨A⟩^2)⟩=⟨A^2⟩-⟨A⟩^2$$`

测不准原理:
`$$⟨(ΔA)^2⟩⟨(ΔB)^2⟩≥\frac{1}{4}|⟨[A,B]⟩|^2.$$`
<!-- <hr>
`$\mathcal{Proof:}$`
Using Schwarz inequality:
`$$⟨(ΔA)^2⟩⟨(ΔB)^2⟩≥|⟨ΔAΔB⟩|^2$$`
`$$ΔAΔB=\frac{1}{2}[ΔA,ΔB]+\frac{1}{2}\{ΔA,ΔB\},$$`
<hr> -->

[`$\leftarrow$`现代量子力学笔记(二)]({{< relref "/notes/mqm-note02.md" >}})[`$\mapsto$`现代量子力学笔记(四)]({{< relref "/notes/mqm-note04.md" >}})
