---
title: [论文总结] OpenCL版Caffe——一个高速跨平台机器学习框架
date: 2017-9-29
---

[Origin article](httpmuratbuffalo.blogspot.hk201709paper-summary-opencl-caffe-accelerating.html)  
作者：Murat Demirbas(httpswww.cse.buffalo.edu~demirbas)  
译者：代亚暄 (httpstagineerdai.github.io)

2016年的[这篇文章](httpdl.acm.orgcitation.cfmid=2909443&dl=ACM&coll=DL)展示了移植到OpenCL的深度学习框架[Caffe](httpcaffe.berkeleyvision.org)。更准确的说，这个版本的Caffe使用开源的标准OpenCL后端，代替了原本的CUDA后端（译者注：Caffe原本的后端CUDA是闭源的）。它一开始公布在一个[独立的GitHub仓库](httpsgithub.comamdOpenCL-caffe)，之后被迁移到了[Caffe的官方GitHub仓库](httpsgithub.comBVLCcaffetreeopencl)中。

开发深度神经网络模型时，我们总是希望尽量降低跨平台部署（服务器，NVIDIA和AMD的显卡，甚至智能手机和平板电脑），和适配不同应用时的迁移成本。然而，包括Caffe在内的大多数深度学习框架都集成了CUDA并仅支持NVIDIA显卡，跨平台迁移受到了局限。

由于各大商业芯片厂商（Altera, AMD, Apple, ARM Holdings, Creative Technology, IBM, Imagination Technologies, Intel, Nvidia, Qualcomm, Samsung, Vivante, Xilinx, ZiiLABS等等）的支持，[OpenCL](httpsen.wikipedia.orgwikiOpenCL)支持异构计算（译者注：指在同一系统中使用一种以上的处理器或核），且httpwww.iwocl.orgwp-contentuploadsiwocl-2016-opencl-caffe.pdf)具有跨平台迁移的能力。为了保证平台兼容性，OpenCL会检查特定的驱动并在运行时编译。

OpenCL最初由Apple开发，之后被转给Khronos Group。它被Android，Linux，FreeBSD，MacOS和Windows在内很多操作系统支持。

### OpenCL的后端移植和优化

Caffe framework is originally written in C++ and CUDA. The CUDA layer of Caffe handles optimization of hardware resource allocation and utilization, e.g. CPU-GPU task assignment, memory management, and data transfer. Since CUDA and OpenCL are different in hardware device abstraction, memory buffer management, synchronization, data transfers, the OpenCL backbend porting is not a straightforward process.

The paper breaks the OpenCL porting process into two phases. Phase 1 achieves a layerwise porting of 3 layers, namely C++ machine learning interfaces, OpenCL wrappers, and GPU kernels. Layerwise porting means the layers are ported one by one and unit tested by using the originals of the other layers to guarantee correctness and convergence of the DNN algorithm.

After all layers are ported to OpenCL in Phase 1, the Phase 2 focuses on performance optimization. Profiling the OpenCL port in Phase 1 (via the AMD profiling tool, CodeXL, assisted with OpenCL event and printf) demonstrates some big bottlenecks. OpenCL's online compilation frequently calls clBuildProgram to create each GPU kernel for 100 iterations of Cifar training, there were 63 clBuildProgram calls that took about 68% of the time. Another bottleneck was that the convolutional layers take up most of the computation time. BLAS's performance suffered from irregular tall and skinny matrix sizes from different layers.

To handle these, the paper proposes three key optimization techniques including kernel caching to avoid OpenCL online compilation overheads, a batched manner data layout scheme to boost data parallelism, and multiple command queues to boost task parallelism. The optimization techniques effectively map the DNN problem size into existing OpenCL math libraries, and improve hardware resources utilization and boost performance by 4.5x.

![evaluation image](http3.bp.blogspot.com-fEbmBtHJZgQWb0u67oYp6IAAAAAAAAGj0TOKcRN7VH8YkI0S52QEDCxt2NbspCqNOgCK4BGAYYCws1600Screen%2BShot%2B2017-09-16%2Bat%2B10.01.50%2BAM.png)

Compared to the highly optimized machine learning cuDNN library, OpenCL Caffe still has a performance gap of 2x as it lacks those optimizations. The authors argue given the current performance, the OpenCL caffe is still competitive in terms of performance per dollar, considering the market price difference of AMD R9 Fury (about 560 dollars) and the NVIDIA TitanX (about 1000 dollars).

### 跨平台性能分析

A natural question that occurs is, would their OpenCL port of Caffe that is tested on AMD GPUs would automatically work with ARM MALI GPUs as well That would be a good test of portability of OpenCL port of Caffe. This has not been answered in the paper.

However, the authors caution about minor problems in compatibility. There are some differences in specific manufacturers' extension and keywords. For example, caffe uses a lot of template in GPU kernels to support different floating point precision. But it turns out the template keywords for different manufactures are different, which adds more difficulty for the same code to run on different platform without modifications.

[OpenCL support of deep learning frameworks](httpsen.wikipedia.orgwikiComparison_of_deep_learning_software) is still not great, but hopefully it is getting better everyday as this paper shows.

The slides presentation for the paper is also available [here](httpwww.iwocl.orgwp-contentuploadsiwocl-2016-opencl-caffe.pdf).
