---
title: OpenCL Caffe
date: 2017-09-21
---

去年，AMD Research将Caffe从CUDA迁移到了OpenCL上。OpenCL版Caffe虽然在性能上仍与cuDNN有差距，但平台兼容性好，在性价比方面优势明显。

---

作者[Murat Demirbas](https://www.cse.buffalo.edu/~demirbas/)，译者[代亚暄](https://tagineerdai.github.io/)。本文已获作者授权，首发于[InfoQ](http://www.infoq.com/cn/)。

点击查看英文原文：[Paper summary. OpenCL Caffe: Accelerating and enabling a cross platform machine learning framework ](http://muratbuffalo.blogspot.hk/2017/09/paper-summary-opencl-caffe-accelerating.html)


2016年的[这篇文章](http://dl.acm.org/citation.cfm?id=2909443&dl=ACM&coll=DL)展示了移植到OpenCL的深度学习框架[Caffe](http://caffe.berkeleyvision.org/)。更准确地说，这个版本的Caffe使用开源的标准OpenCL后端，代替了原本的CUDA后端（译者注：Caffe原本的后端CUDA是闭源的）。它一开始公布在一个[独立的GitHub仓库](https://github.com/amd/OpenCL-caffe)，之后被迁移到了[Caffe的官方GitHub仓库](https://github.com/BVLC/caffe/tree/opencl)中。

开发深度神经网络模型时，我们总是希望尽量降低跨平台部署（服务器，NVIDIA和AMD的显卡，甚至智能手机和平板电脑），和适配不同应用时的迁移成本。然而，包括Caffe在内的大多数深度学习框架都集成了CUDA并仅支持NVIDIA显卡，跨平台兼容性受到了局限。

由于各大商业芯片厂商（Altera, AMD, Apple, ARM Holdings, Creative Technology, IBM, Imagination Technologies, Intel, Nvidia, Qualcomm, Samsung, Vivante, Xilinx, ZiiLABS等等）的支持，[OpenCL](https://en.wikipedia.org/wiki/OpenCL)支持异构计算（译者注：指在同一系统中使用一种以上的处理器或内核）具有跨平台迁移的能力。为了保证平台兼容性，OpenCL会检查特定的驱动并在运行时编译。

OpenCL最初由Apple开发，之后被转给Khronos Group，它被Android、Linux、FreeBSD、MacOS和Windows在内的很多操作系统支持。

---

### OpenCL的后端移植和优化

最初的Caffe是用C++和CUDA写成的。Caffe的CUDA层负责优化硬件资源分配和使用，比如CPU/GPU间任务调度、内存管理和任务传输。由于CUDA和OpenCL在设备抽象、缓存管理、同步处理和数据传输的实现上的差别，从CUDA到OpenCL后端的迁移并没有看上去那么简单。

这篇文章将向OpenCL移植的过程划分成两个阶段。第一阶段是C++机器学习接口、OpenCL封装器和GPU内核这三个层的逐层移植。逐层移植，意味着每个层会被依次移植，并在其他层都为原CUDA层的环境下进行单元测试，以确保深度神经网络算法的正确性和收敛性。

第一阶段完成后所有的层都已经移植到OpenCL下，第二阶段关注的是性能提升。通过AMD分析工具、CodeXL、以printf结合OpenCL事件进行信息输出等方法的性能分析显示，完成第一阶段到OpenCL的移植后，还存在一些大的性能瓶颈。OpenCL的在线编译器会频繁调用clBuildProgram来创建GPU内核——训练Cifar数据集的100次迭代中，clBuildProgram就调用了63次，占用总运行时间多达68%；另一个瓶颈在于，卷积层占用了大多数计算时间。由于不同层间矩阵形状不规则（矩阵长宽比过大），BLAS的效果相当差。

为解决上述问题，这篇文章提出了三个关键优化技术。GPU内核使用高速缓存，可以避免OpenCL在线编译器负荷过高；使用批处理的数据布局方案，可以提升数据并行化；使用多个队列进行任务处理，可以提升任务并行化。这些优化技术有效地将深度神经网络问题的规模，映射到现有的OpenCL数学库上，通过优化硬件资源的利用率，将性能提升了4.5倍左右。

![evaluation image](https://raw.githubusercontent.com/TagineerDai/blog/master/source/_misc/OpenCL_Performance.png)

OpenCL版Caffe由于优化不完整，和已经进行过较彻底优化的机器学习库cuDNN相比，目前还有2倍左右的性能差距。作者提出，虽然性能上存在差异，但当我们将性价比（每美元带来的性能效果）纳入考量时，OpenCL版Caffe的优势就显示出来了——AMD R9 Fury的市价大概560美元，而NVIDIA TitanX在一千美元左右。

---

### 跨平台性能分析

我们很自然地会想到另一个问题，经过了在AMD上的测试，OpenCL版Caffe是否也能直接在ARM的MALI上工作良好呢？这方面的测试可以很好地反映OpenCL版Caffe的兼容性，然而并没有在这篇文章中提及。

不过，作者们确实注意到了兼容性方面的一些细节问题，文中提到：“特定厂商的扩展名和关键字有一些差异。例如，Caffe使用了大量的GPU内核模板来支持不同的浮点精度。但不同厂家使用的模板关键字是不同的，这增加了相同代码不经修改在不同平台运行的难度。”

[OpenCL对深度学习框架的支持](https://en.wikipedia.org/wiki/Comparison_of_deep_learning_software)还不完美，但好在，就像我们在这篇文章中看到的，情况正在逐渐得到改善。

本文所讨论的论文对应的PPT可从[此处](https://github.com/TagineerDai/blog/blob/master/source/_misc/iwocl-2016-opencl-caffe.pdf)获得。
