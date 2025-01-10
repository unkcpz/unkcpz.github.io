+++
title = "现代量子力学笔记(七): Shrödinger的画和Heisenberg是一样的(OVA2)"
Tags = ["note", "physics", "quantum"]
Categories = ["note"]
date = 2019-01-03
+++

## 结果对比

|            | Schrödinger picture | Heisenberg picture |
|------------|----------------------|--------------------|
| State ket  | Moving               | Stationary         |
| Observable | Stationary           | Moving             |
| Base ket   | Stationary           | Moving oppositely  |

在Schrödinger picture中,算符对应的观测量是不随时间变化的,基矢也是不随时间变化的.
随时间变化的只有一定时刻下的态.
随时间变化的关系受含时Shrödinger方程约束:
`$$iħ\frac{∂}{∂t}|α,t_0;t⟩=H|α,t_0;t⟩,$$`

在Heisenberg picture中,某一时刻的状态是不随时间变化的,变化的是算符和基矢.
观测量由算符描述,随时间的变化关系如下:
`$$\frac{dA^{(H)}}{dt}=\frac{1}{iħ}\left[A^{(H)},H \right],$$`

同时基矢(Base ket)满足如下随时间变化的关系,称为 *wrong-sign Shrödinger equation*:
`$$iħ\frac{∂}{∂t}|a',t⟩_H=-H|a',t⟩_H.$$`

## 以上关系的来源
unitary算符在为位移算符或时间演化算符时,会让一个体系的态发生变化.
观察`$⟨β|X|α⟩$`如何随着该算符变化可以得到:
`$$(⟨β|U^†)⋅X⋅(U|α⟩)=⟨β|⋅(U^†XU)⋅|α⟩.$$`

上式等好左右两边分别是两种看待变换的方式,左边的unitary算符作用在ket上,使得态发生了变化.
右边的unitary算符作用在算符X上,而当前态本身没有发生变化.

第一种方式就是Shrödinger picture第二种是Heisenberg picture, 其中Heisenberg picture描述了
算符随时间的变化,这和经典力学中的变化描述是相同的.

## Ehrenfest's Theorem
`$$m\frac{d^2}{dt^2}⟨\mathbf{x}⟩=\frac{d⟨\mathbf{p}⟩}{dt}=-⟨∇V(\mathbf{x})⟩.$$`

...
