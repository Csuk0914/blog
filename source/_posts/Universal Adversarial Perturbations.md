---
title: Universal adversarial perturbations
date: 2017-10-23
categories: translation
---

对输入图片进行微小的扰动，就可导致网络做出错误判断。这篇论文中描述了全局扰动的计算过程，并对其泛化和迁移能力，信息含量，以及理论可行性进行了简单介绍。

作者[Adroam-colyer](https://twitter.com/adriancolyer)，译者[代亚暄](https://tagineerdai.github.io/)。本文已获作者授权，同时发布于[InfoQ](http://www.infoq.com/cn/)。

点击查看本文英文原文：[Universial adversarial perturbations](https://blog.acolyer.org/2017/09/12/universal-adversarial-perturbations/)，以及[同名相关论文](https://arxiv.org/abs/1610.08401)。

---

对抗性扰动非常有趣，对深度网络的输入施加微小的变化，就会导致网络做出错误的判断。

我们在前些时间对对抗性图像的研究表明，所有参数数目足够大的深度网络似乎都很脆弱，并且似乎现在还没有什么抵抗对抗性图像的方法。

今天的这篇文章，以生成能够使得特定的输入图像被错误分类的扰动为目标，为我们展示了单一扰动是如何带来大范围的图像错误分类的。认识到模型的弱点，能够帮助我们更好地理解它们如何工作。

在二十世纪的战争中，我们通常使用伪装使得观察者错误地对看到的东西作出判断。（比如“那不是坦克，只是一些植被”。）

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-1.jpeg)


二十一世纪，如果我们最终进入了某种AI形式的战争（很不幸，似乎从某种程度上讲这是不可避免的），用于迷惑敌军AI系统的＂迷彩＂或许长这个样子：
![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-2.jpeg)

观看[这个视频](https://www.youtube.com/watch?v=jhOu5yhe0rc&feature=youtu.be)来看看动画版。（我也不太清楚自己为什么要做这么惨的比喻......我们还是看回图片吧）

想搞清楚普遍扰动的工作原理，要看下下图，左边展示了正确分类的图片，在都加入中间图片的扰动后得到右侧图片，大部分被分错了了类别。

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-3.jpeg)

对人眼来说，这些扰动很难观察到。这是一系列加了扰动的图片——如果不是不寻常的标签值（被错误地分类了），第一眼看上去你很难发现有何区别。但如果你离近些看的更仔细，就能找到有规律的螺旋花纹。比如，最突出的就是被错认为三趾树懒（three toed sloth）的骆驼图片的蓝天背景。

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-4.jpeg)

([点击查看大图](https://adriancolyer.files.wordpress.com/2017/09/uap-fig-3.jpeg))

---

### 如何计算全局扰动向量

给定一个分类器（这篇论文中是自然图片分类器），目标是寻找一个扰动向量`v`，当`v`被加在任意图片输入上的时候，分类器会以大于`1-δ`的概率预测出错误的标签。同时，由于希望`v`给原图带来的改动尽量小，我们会增加如下限制：

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-Formula-0.JPG)

> 参数`ε`控制着扰动向量`v`的大小，`δ`量化了图片在扰动前后的迷惑率（fooling rate）。

扰动向量是迭代发展的。在每一次迭代中，我们采样一个数据点`x_i`，如果当前的普遍扰动`v`不能误导分类器将`x_i`分错，我们就寻找其极小范数能够迷惑分类器的额外扰动`δv_i`。当扰动后数据上的迷惑率超过目标阈值时，该算法终止。

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-5.jpeg)

虽然算法只随机求得一个扰动图像，但它可以被用于生成针对给定神经网络的多个全局扰动。

下图展示了算法如何工作：
![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-6.jpeg)

---
### 常见深度网络的全局扰动计算

用上面介绍的方法，在ILSVRC 2012数据集上训练学习全局扰动，然后在有50,000张图像的，没有用于扰动训练的验证集上，测试多少图像由于分类器被扰动误导而分错了。作者统一使用ILSVRC2012的数据，在几个最近的深度神经网络上都进行了重复实验。

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-7.jpeg) 

>我们发现对所有的网络来说，全局扰动都能为验证集带来非常高的迷惑率。特别是VGG-F和CaffeNet上的结果，验证集上的迷惑率达到了90%以上。

不同网络结构上求得的扰动图像如下：

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-8.jpeg) 

>算法（Algorithm 1）求得的全局扰动在数据上有非常强的泛化能力，且只需在一个非常小的训练集上学习就可达到这种效果。

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-9.jpeg) 

---

### 全局扰动可以跨网络结构迁移么？

我们现在可以为任何模型构建全局扰动，但一个模型上构造的扰动是否可以迷惑其他模型呢？在相当高的程度上，是的！在VGG-19上训练出的全局扰动，在其他网络结构上测得的迷惑率大约53%。

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-10.jpeg) 

>上面的结果显示了不同数据和网络结构上泛化性能良好，证明这些扰动间存在实际联系的。特别地，简单地对一张从未见过的图片叠加VGG-19上训练得到的全局扰动，其他网络结构错分的几率就非常高。

### 扰动后的图片做fine-tuning可以提高模型能力么？

如果你用加了扰动的图片继续训练分类器，确实可以将迷惑率稍稍降低。比如，在VGG-F上经过5个迭代的扰动后图片fine-tuning训练后，分类的错误率从93.7%降到了76.2%。然而，不考虑迭代次数，增加新的扰动后数据，重复这个过程并不能带来进一步的提升，错误率会在80%上下浮动。

### 全局扰动为何有效？

以点表示预测标签，以从点`i`到点`j`的有向边表示大部分真实标注为i的图片被错误地标注为了`j`，这样构建出的图含有特殊的拓扑结构。

>特别的是，当一个联通块中所有的边都指向一个目标标签，构建出的图就是一些不相连的联通子图的交集。

比如，在下面的结构中，许多枕头就被错误地预测成了乌林鸮。

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-11.jpeg) 

>我们假设这些主导标签占据了图像空间中的大部分区域，那么它们自然是迷惑自然图像分类器的优质的候选标签。

在另一个实验中，作者对比了全局扰动和随机扰动在迷惑率上的区别，发现全局扰动效果好的出奇。

>全局和随机扰动之间的巨大差别揭示了，全局扰动发掘出了分类器决策边界不同部分间的几何相关性。

通过探索正交于给定输入图像在分类器中决策边界的向量，作者证明在给定自然图像的周围，有可能存在低维度`d'`上的子空间`S`，包含正交于决策边界的大多数向量。事实上，从这个子空间中选择一个随机向量，相比于无约束地随机选择扰动向量，能够显著提高迷惑率（从10％到38%）。

>下图展示了描述决策边界相关性的子空间`S`。有这种低维空间存在，说明全局扰动有很好的泛化性能，通过很少的图片训练的扰动向量就可以达到不错的效果。 

![](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/Adv-Pert-12.jpeg)

[这个视频](https://www.youtube.com/watch?v=jhOu5yhe0rc&feature=youtu.be)展示了手机平台上运行的全局扰动算法。
