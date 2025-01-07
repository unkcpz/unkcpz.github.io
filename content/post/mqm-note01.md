+++
title = "现代量子力学笔记(一): 实验和意念显现"
Tags = ["note", "physics", "quantum"]
Categories = ["note"]
date = 2018-12-18
+++

该部分笔记与Sakurai现代量子力学(1.1)对应

## Stern-Gerlach 实验
这个实验直观表示出,一个原子尺度的系统会表现出量子的特性.这种所谓的量子的特性,
是与经典物理的直觉相违背的.

实验的装置如下:
![](https://upload.wikimedia.org/wikipedia/commons/e/ee/Stern-Gerlach_experiment_svg.svg)
(Wiki图)银原子经过特定方向的磁场,在探测设备上得到该银原子的落点; (1) furnace, (2) beam of silver atoms,
(3) inhomogeneous magnetic field, (4) classically expected result, (5) observed result

### 第一个和经典相违背的结果

在这个实验中,可以得到一个与经典直觉相违背的结果:发射出的银原子因为角动量的方向各异本应该在观测器上得到连续的落点(上图[4]),
但结果只观测到分立的两个落点(上图[5]).

这说明,出射的银原子的角动量不是如同经典所理解的有方向变化的是分立的,而是如所观测的离散的.

## Stern-Gerlach Sequential 实验
通过上面的实验,原子束会被分成分立的两束,通过串联这样的装置可以进行下面这样的实验:
![](https://upload.wikimedia.org/wikipedia/commons/3/35/Sg-seq.svg)

(top) 最上方的系统表示,经过分离z方向S-G装置(z-SG)粒子束被分成`$z_{-}$`和`$z_{+}$`两束,过滤掉`$z_{-}$`,
仅让`$z_{+}$`再次通过z-SG,仅仅得到`$z_{+}$`,`$z_{-}$`完全被过滤了.
这是直觉可以接受的.

(middle) 中间的系统表示,经过分离z方向S-G装置(z-SG)粒子数被分成`$z_{-}$`和`$z_{+}$`两束,过滤掉`$z_{-}$`,
仅让`$z_{+}$`通过分离x方向S-G装置 (*x-SG*),可以得到`$x_{-}$`和`$x_{+}$`.
x方向上的信息都有,这从直觉上也没什么问题.

(bottom) 同上方装置,但在最后又过滤了`$x_{-}$`直留下`$x_{+}$`经过后续的(*z-SG*).发现此时粒子束,
又被分成了`$z_{-}$`和`$z_{+}$`两束.
这是和直觉相违背的,已经被过滤干净的`$z_{-}$`怎么又在后面重新出现了呢?

### 第二个和经典相违背的结果
Stern-Gerlach Sequential实验的最后一个结果与经典直觉所理解的不同,为什么一个已经被过滤的信息有重新出现了呢?

这个例子在量子力学中被用来说明我们不能同时(simultaneously)确定z方向上的结果和x方向上的结果.
更详细的说就是,粒子束在进入一个x-SG装置后,其中z方向上的信息就被破坏了.

## Stern-Gerlach结果与经典光学的类比
上面的结果使我们想到经典光学中类似的情况:
沿z方向传播,沿x方向偏振的单色光为:
`$$\mathbf{E}=E_0 \mathbf{\hat{x}} \cos(kz-\omega t)$$`
沿z方向传播,沿x方向偏振的单色光为:
`$$\mathbf{E}=E_0 \mathbf{\hat{y}} \cos(kz-\omega t)$$`

使用偏振片可以将一束同时含有x和y方向的光分成分别沿着x方向和y方向偏振的光.
在这里偏振片的效果如同SG实验中的一个SG装置.

与上面的串联的SG实验类比,当x偏振片后,出射光只剩下沿着x方向振动的部分,而没有y方向振动的部分.
如何重新获得这一部分? 我们使用沿着x和y方向夹角45度的方向放置一个偏振片,使得光沿着`$\mathbf{\hat{x}}^{'}$`
方向偏振,则再次经过y方向的偏振片后便又能重新获得沿y方向的偏振光.

而如果不经过沿着45度方向的偏振片,直接通过y方向的偏振片,是无法获得沿着y方向的偏振光的.

我们可以做如下完整的类比:
`$$S_{z}\pm \iff x-, y- \mathrm{polarized\  light}$$`
`$$S_{x}\pm \iff x'-, y'- \mathrm{polarized\  light}$$`
(fig)

沿着`$\mathbf{\hat{x}}^{'}$`和`$\mathbf{\hat{y}}^{'}$`方向的光使用向量的组合方式可以写成如下形式:
`$$E_0 \mathbf{\hat{x}}'\cos(kz-\omega t)=E_0\left[\frac{1}{\sqrt{2}}\mathbf{\hat{x}}\cos(kz-\omega t)+\frac{1}{\sqrt{2}}\mathbf{\hat{y}}\cos(kz-\omega t) \right]$$`
`$$E_0 \mathbf{\hat{y}}'\cos(kz-\omega t)=E_0\left[-\frac{1}{\sqrt{2}}\mathbf{\hat{x}}\cos(kz-\omega t)+\frac{1}{\sqrt{2}}\mathbf{\hat{y}}\cos(kz-\omega t) \right]$$`

对应到Stern-Gerlach实验,我们将结果写为如下形式以完成类比:
`$$|S_x;+\rangle=\frac{1}{\sqrt{2}}|S_z;+\rangle + \frac{1}{\sqrt{2}}|S_z;-\rangle$$`
`$$|S_x;-\rangle=-\frac{1}{\sqrt{2}}|S_z;+\rangle + \frac{1}{\sqrt{2}}|S_z;-\rangle$$`

这里漏掉了S-G实验中的`$S_y$`的情况,如何表示`$S_y$`?这个分量对`$S_z$`的行为和`$S_x$`是等同的,
但三者互相之间又是不同的.
在经典光学中引入圆偏振光,对`$S_y$`进行类比:
`$$S_{y}+\iff \mathrm{right\  circularly\  polarized\  light}$$`
`$$S_{y}- \iff \mathrm{left\  circularly\  polarized\  light}$$`

左旋和右旋的偏振光可以写为:(引入复数的写法)
`$$\mathbf{\epsilon}=\left[\frac{1}{\sqrt{2}}\mathbf{\hat{x}}e^{i(kz-\omega t)} \pm \frac{i}{\sqrt{2}}\mathbf{\hat{y}}e^{i(kz-\omega t)} \right]$$`

回到S-G实验,可以得到`$S_y$`如下:
`$$|S_y;\pm\rangle=\frac{1}{\sqrt{2}}|S_z;+\rangle \pm \frac{i}{\sqrt{2}}|S_z;-\rangle$$`

## 总结
从以上的实验可以看到与经典不同的结果,让我们开始有必要用到新的手段来描述问题.

有经验的读者可能会感觉用光来对应实验,与其说是类比,不如说是完全等同,因为由于波粒二向性,光可以用光子来描述.
实际上,上面的类比没有用到光子(photon)的概念,完全使用的是经典的电磁波理论.

这样的实验和类比引导我们使用新的工具来研究微观的新现象.
下一节则描述了描述的工具和数学方法.

[`$\mapsto$`现代量子力学笔记(二): 意念具象化?数学和符号!]({{< relref "/notes/mqm-note02.md" >}})
