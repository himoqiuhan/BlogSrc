---
title: 技术美术百人计划学习笔记（图形2.9 GPU硬件架构及运行机制）
date: 2023-06-14 12:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 深入GPU硬件架构及运行机制
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true 
---

![【图形渲染】 2.9 GPU硬件架构及运行机制](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2900.png)

## 前言问题及快速索引

先是我课前的回答，然后再等上课时或者上课后修正

1. GPU是如何与CPU协调工作的？

   [CPU与GPU的异构系统](##CPU-GPU的异构系统)

2. GPU也有缓存机制吗？有几层？他们的速度差异多少？

   [GPU内存架构](####7.1 GPU内存架构)

3. GPU的渲染流程有哪些阶段？他们的功能分别是什么？

   [GPU渲染流程](####2.2 完整渲染流程)

4. Early-Z技术是什么？发生在哪个阶段？这个阶段还会发生什么？

   [Early-Z](##3 Early-Z)

5. SIMD和SIMT是什么？他们的好处是什么？co-issue呢？

   [SIMD](####SIMD)、[SIMT](####SIMT)、[co-issue](####co-issue)

6. GPU是并行处理的吗？若是，硬件层是如何设计和实现的？

   [CPU与GPU的对比](##CPU与GPU的对比)、[GPU架构的共性](####GPU架构的共性)

7. GPC、TPC、SM是什么？Warp又是什么？它们和Core、Thread之间的关系如何？

   [GPU架构的共性](####GPU架构的共性)

8. 顶点着色器（VS）和像素着色器（PS）可以是同一处理单元吗？为什么？

   可以，用的都是Core

9. 像素着色器（PS）的最小处理单元是1像素吗？为什么？会带来什么影响？

   [渲染流程14](####2.2 完整渲染流程)

10. Shader中if、for等语句会降低渲染效率吗？为什么？

    [渲染流程14](####2.2 完整渲染流程)

11. 如下图，渲染相同面积的图形，三角数量少（左）的还是数量多的（右）的效率更快？为什么？

    ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2900-1.png)

    - 答案是：同一个像素块如果分属不同的三角形，就会分配到不同的SM进行处理。由此推断，相同面积的区域，如果所属的三角形越多，就会导致分配给SM的次数越多，消耗的渲染性能也越多。

12. GPU Context是什么？有什么作用？

    [GPU Context](##9 GPU Context和延迟)

13. 造成渲染瓶颈的问题很可能有哪些？该如何避免或优化他们？

    [总结及渲染优化建议](10 总结及渲染优化建议)

    1. 数据传输的带宽问题：优化模型、压缩贴图
    2. CPU切换渲染状态过于频繁：合并使用同一材质的模型，减少场景中的材质总数
    3. GPU性能限制：降低画质

## GPU硬件架构

#### GPU是什么

GPU全称Graphics Processing Unit，图形处理单元。

是专门用于绘制图像和处理图元数据的特定芯片

![NVIDIA GPU芯片实物图](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/471fb59b-e239-45a7-a861-61cffb1d35cb.jpeg)

NVIDIA GPU芯片实物图

#### GPU物理架构

- 由于纳米工艺的引入，GPU可以将数以亿计的晶体管和电子器件集成在一个小小的芯片内。从宏观物理结构上看，现代大多数桌面级GPU的大小跟数枚硬币等同大小，部分甚至比一枚硬币还小。

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/3705924a-5482-4a7e-8011-1f2eaa9e38df.png)

- 当GPU结合散热风扇、PCI插槽、HDMI接口等部件之后，就组成了显卡

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/21140ab7-1507-48de-9284-b1c19032c9ea.png)

- 显卡不能独立工作，需要装载在主板上，结合CPU、内存、显存等硬件设备组成完整的PC机

#### GPU硬件架构

##### NVIDIA GPU架构发展关键点

- 2008 - Tesla
  - Tesla最初是给计算处理单元使用的，应用于早期的CUDA系列显卡芯片中，并不是真正意义上的普通图形处理芯片。
- 2010 - Fermi
  - Fermi是第一个完整的GPU计算架构。首款可支持与共享存储结合纯cache层次的GPU架构，支持ECC的GPU架构。
- 2014 - Maxwell
  - 其全新的立体像素全局光照 (VXGI) 技术首次让游戏 GPU 能够提供实时的动态全局光照效果。基于 Maxwell 架构的 GTX 980 和 970 GPU 采用了包括多帧采样抗锯齿 (MFAA)、动态超级分辨率 (DSR)、VR Direct 以及超节能设计在内的一系列新技术。
- 2018 - Tering
  - Turing 架构配备了名为 RT Core 的专用光线追踪处理器，能够以高达每秒 10 Giga Rays 的速度对光线和声音在 3D 环境中的传播进行加速计算。Turing 架构将实时光线追踪运算加速至上一代 NVIDIA Pascal™ 架构的 25 倍，并能以高出 CPU 30 多倍的速度进行电影效果的最终帧渲染。2060系列、2080系列显卡也是跳过了Volta**直接选择了Turing架构。**

##### NVIDIA Tesla架构

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/cd66d1dc-5f85-4259-80de-6d1cd1c8f14b.png)

- 拥有7组TPC (Texture Process Cluster, 纹理处理簇)
- 每个TPC有两组SM (Stream Multiprocessor — 流多处理器)
- 每个SM包含：
  - 8个SP (Streaming Processor — 流处理器)
  - 2个SFU (Special Function Unit — 特殊函数单元)
  - L1缓存、MT Issue（多线程指令获取）、C-Cache（常量缓存）、共享内存
- 除了TPC核心单元，还有与显存、CPU、系统内存交互的各种部件

##### NVIDIA Fermi架构

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/b9966bc7-2ca2-4fa7-bb96-279eb2f9e35f.png)

- 拥有16个SM
  - 2个Wrap Schedual（线程束）
  - 两组共32个Core
  - 16组加载存储单元（LD/ST）
  - 4个特殊函数单元（SFU）
  - 分发函数（Dispatch Unit）
- 每个Core：
  - 1个FPU（浮点数单元）
  - 1个ALU（逻辑运算单元）

##### NVIDIA Maxwell架构

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/ce61c7b2-8a10-4509-81b0-7372b4d8b991.png" alt="" style="zoom:80%;" />

- 采用Maxwell的GM204拥有4个GPC（Graphics Process Cluster — 图形处理簇）
- 每个GPC有4个SM，对比Tesla架构来说，在处理单元上有了很大的提升

##### NVIDIA Turning架构

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/11c0c83b-1c03-459d-b753-1ad1d843fe28.png" alt="" style="zoom: 25%;" />

- 6个GPC（图形处理簇）
- 36个TPC（纹理处理簇）
- 72个SM（流多处理器）
- 每个GPC有6个TPC，每个TPC有2个SM
- 4608个CUDA核
- 72个 RT核
- 288个纹理单元
- 12x32位GDDR6内存控制器（共384）

其中每个SM的结构如下：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/024272b6-85bb-4f1b-a3ab-6a03f52bf28e.png" alt="" style="zoom: 50%;" />

- 64个CUDA核
  - CUDA是NVIDIA推出的统一计算架构
- 8个Tensor核
  - Tensor Core是专门为执行张量或矩阵运算而设计的专用执行单元
- 256KB寄存器文件
- RT Core

##### GPU架构的共性

- GPC：图形处理蹙
- TPC：纹理处理簇
- Thread：线程
- SM、SMX、SMM（Stream Multiprocessor，流多处理器）
- Warp线程束、Warp Scheduler（Wrap编排器）
- SP（Streaming Processor，流处理器）
- Core（执行数学运算的核心）
- ALU（逻辑运算单元）
- FPU（浮点数单元）
- ROP（Render Output Unit，渲染输出单元）
- Load/Store Unit（加载存储单元）
- L1 Cache（L1缓存）
- L2 Cache（L2缓存）
- Shared Memory（共享内存）
- Register File（寄存器）

GPU为什么会有那么多层级且有那么多雷同的部件？因为GPU的任务是天然并行的，现代GPU的架构皆是以**高度并行能力**而设计的

核心组件结构：

- 包含关系：GPC→TPC→SM→CORE
- SM中包含Poly Morph Engine（多边形引擎）、L1 Cache（L1缓存）、Shared Memory（共享内存）、Core（执行数学运算的核心等）
- CORE中包含ALU、FPU、Execution Context（执行上下文）、Detch、Decode（解码）

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/7dcde62d-8d2c-49d3-8c48-56ffb9219178.png)

## GPU的运行机制

#### GPU渲染总览

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/678d15f1-0a7c-4e5f-86b8-dd155f234197.png" alt="Fermi架构运行机制总览" style="zoom: 50%;" />

Fermi架构运行机制总览

从Fermi开始NVIDIA使用类似的原理架构，使用一个Giga Thread Engine来管理所有正在进行的工作，GPU被划分成多个GPCs（Graphics Processing Cluster），每个GPC拥有多个SM（SMX、SMM）和一个光栅化引擎（Raster Engine）。他们其中有很多的连接，最显著的是Crossbar，它可以连接GPCs和其他功能性模块（例如ROP或其他子系统）

程序员编写的shader是在SM上完成的，每个SM包含许多为线程执行数学运算的Core（核心）。例如，一个线程可以是顶点或像素着色器驱动，Warp Schedule管理一组32个线程作为Warp（线程束），并将要执行的指令移交给Dispatch Units

GPU中实际有多少这些单元（有多少个GPC，每个GPC有多少个SM……）取决于芯片配置本身

#### 完整渲染流程

1. 首先，场景中的模型包含了mesh数据，位置信息等，经过camera的粗粒度裁剪获得真正需要显示在屏幕中的模型。程序通过图形API（DX、GL、WEBGL）发出drawcall指令，指令会被推送到程序驱动，驱动会检查指令的合法性，然后会把指令放到GPU可以读取的PushBuffer中
2. 经过一段时间或者显式调用flush指令后，驱动程序把PushBuffer的内容发送给GPU，GPU通过主机接口（Host Interface）接受这些命令，并通过前端（Front End）处理这些命令
3. 在图元分配器（Primitive Distributor）中开始工作分配，处理IndexBuffer中的顶点产生三角形分成批次（batches），然后发送给多个GPCs。这一步的理解就是提交上来n个三角形，分配给这几个GPC同时处理。

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/2be8e11c-e5ab-4cc7-af39-81180307b26a.png)

这一阶段的理解就是场景中的数据包括vbo，cbuffer等已经提交到GPU，GPU通过指针绑定到对应的数据层面上通过GPC开始同时处理。

4. 在GPC中，每个SM中的Poly Morph Engine负责通过三角形索引（Triangle Indices）取出三角形的数据（Vertex Data），即下图中的Vertex Fetch模块

   ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2900-12.png)

5. 在获取数据之后，SM会以32个线程（Thread）为一组，组成一个线程束（Wrap），开始处理顶点的数据

   - Wrap是典型的**单指令，多线程（Single instruction, multiple thread - SIMT）**，32个线程同时执行的指令是一模一样的，只是线程数据不一样，这样的好处就是一个Wrap只需要一套逻辑，对指令进行解码、执行即可，芯片可以做得更小更快，因为GPU处理的任务是天然并行的

   - SIMT与SIMD(单指令多数据)的差别：简而言之就是输入一个vec4进行计算时，SIMT会把vec4拆分为四个float进行计算，执行4次cycle，而SIMD则是进行一次计算，执行1次cycle。在负责计算的情况下，SIMT相比于SIMD，对开发者和编译器的要求更低，更好优化
     资料：[https://zhuanlan.zhihu.com/p/113360369](https://zhuanlan.zhihu.com/p/113360369)

       1. 从一个线程角度看。从我的研究看，SIMD一般是这样实现的，一个线程处理一条指令，这条指令是向量化处理的。例如一个32bit位宽的4维向量vec4，一条指令最快就在一个cycle执行完。那SIMT，最快要用4个cycles来完成。在SIMT的架构上，会把vec4分解开，然后一个cycle处理完一个数据。所以最快需要4个cycle。好了这里我们讲的是一个线程的情况看，但从这个角度，大家可能都觉得SIMD效率更高。

       2. 单个线程的SIMD核SIMT逻辑单元对比。从上述中看，基本可以认为单个线程看，SIMD相对SIMT需要4倍的逻辑单元。这里的逻辑单元可以认为是最基本的逻辑计算单元。当然也可以理解为单个线程的面积SIMD基本接近SIMT的4倍面积。

       3. 单核同时多线程SIMD和SIMT同等算力对比。很显然SIMD的线程个数是SIMT线程的n分之一就可能实现同等算力。为了达到这种情况，算法的实现必须严格的按照n维的整数倍来实现。

       4. SIMD在线程越来越多的时候不在有优势。很显然，随着线程越来越多，SIMD如果单纯把向量维度增加的话，会出现vec16。对于这么长维度的话，浪费可能越来越多。因此有些架构可能会对16个线程分成4组的SIMD。然后线程打包成4组一个包。

       5. SIMT在多线程的不足。因为SIMT始终是同一条指令的，从寄存器角度看就是PC指针始终是一样的。如果线程太多的时候，有些线程需要分支跳转（if-else），那么效率就会降低，所以很多GPU对分支计算一般耗时是多个分支的总和。

6. SM的warp调度器会按照顺序分发指令给整个warp，单个warp中的线程会锁步（lock-step）执行各自的指令（同一时间执行相同的指令） ，如果线程碰到不被执行的情况也会被遮掩（be masked out）

   - 被遮掩的原因有很多，比如当前的指令是if-true的分支，但是当前线程的数据条件是false；或者循环的次数不一样，比如说for的循环次数N不是一个常量，他是一个动态被计算出来的量，或是被break提前终止但是别的线程还在走。因此，shader中分支会显著增加时间消耗，在一个warp，32个线程中，除非都走到true或false力，否则相当于所有的分支都走一遍。线程不能独立执行指令，而是以一个warp中32个线程为单位，只有每个warp间才是独立的。

   - 可以这样理解：在shader执行过程中，如果这一片32个线程，同时存在true或是false的情况，由于这32个线程是锁步的，那么true和false这两个指令他都会依次执行，只不过它的计算结果会被他的条件所遮蔽。更进一步，此为动态分支；那么静态分支呢？就是同一个warp中32个线程所有都走到同一个分支，即如果我们确保这32个分支里面所有都是true或是所有都是false的情况的话，那么他只会走单个分支。在这个分支执行完毕后，继续执行下一个指令。

   - 变体的执行状况其实和if的判断情况比较相似，比如说我们要执行变体一中的方法，相当于就是一个if true，这时候我们其实可以把它改造成使用if shader中的一个属性或者公共变量来做判断，这样可以减少变体的数量。比如说shader的属性中设置一个变量它的范围是0~1，对应我们的一个变体， 然后我么写一个判断：if (x < 1)，当我们把他设置成0的时候，我们就可以保证所有执行这个shader的线程中都是true，那么我们能够使得线程全部只执行true中的指令，从而获得使用变体相同的效果，这样就可以减少变体数量。

7. 而现在使用的URP管线中，他能否使用SRP Batch，关键就是相同的shader中它使用的变体是否一致

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2900-13.png)

8. warp中的指令可以被一次完成，也可能经过多次调度。

9. 由于某些指令比其他指令需要更长的时间才能完成，特别是内存加载，warp调度器可能会简单地切换到另一个没有内存等待的warp，这是GPU如何克服内存读取延迟的一个关键——只是简单地切换活动线程组

   - 为了加快这种切换，调度器管理的所有warp在寄存器文件中都有自己的寄存器。此处就会产生一个矛盾：如果说当我们的shader中寄存器需要的越多，就会给warp留越少的空间，导致产生越少的warp，这个时候碰到内存延迟，就只能等待，而没有可以运行的warp去切换

   - 猜测这有些游戏GPU没吃多少但帧率就是低是因为shader里寄存器用太多了？

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/88904ac5-40a1-4534-bdb5-0bf8bdd55e8a.png" alt="2.9-14" style="zoom:80%;" />

10. 一旦warp完成了vertex-shader的所有指令，运算结果会被Viewport Transform模块处理，三角形会被裁减然后准备栅格化，GPU会使用L1和L2缓存来进行vertex-shader和pixel-shader的数据通信
    - Viewport Transform模块的处理就是将数据从NDC变换到屏幕空间，并在此过程中进行裁剪

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/a6f08863-e0a6-45fc-93c1-defd1859cae0.png" alt="2.9-15" style="zoom:80%;" />

11. 接下来这些三角形会被分割，再分配给多个GPC，三角形的范围决定它将被分配到那个光栅引擎（Raster Engines），每个Raster Engines覆盖了多个屏幕上的tile，这等于把三角形的渲染分配到了多个tile上面。也就是像素阶段就把三角形进行了按显示的像素的划分，即把显示的数据由按三角形划分变为按显示的像素划分

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/3270af08-6537-4145-9ab4-2c06bda94b8d.png" alt="2.9-16" style="zoom:80%;" />

12. SM上的Attribute Setup保证了从vertex shader来的数据经过插值后是pixel shader可读的

13. GPC上的光栅引擎（Raster Engines）在他接收到的三角形上来工作，来负责这些三角形的像素信息生成（同时会处理背面剔除和Early-Z剔除）

14. 32个像素线程被分成一组**8个2x2的像素块**（这是像素着色器上的最小工作单元，即4个像素），在这个像素线程内，如果没有被三角形覆盖就会被遮掩，SM中的warp调度器会管理像素着色器的任务。
    - **像素着色器最小工作单元是**4个像素的原因是，2x2的像素块我们可以轻易地获取2个像素之间的ddx和ddy，用于计算读取哪个层级的mipmap，从而减少我们从贴图中读取贴图数据的大小，进而降低带宽
    
    - 说起mipmap，开启之后不只是增加内存，让远处部分不会有像素闪烁，因为每个层级的mipmap都是上一层及经过双线性插值或者三线性插值后的结果，其表现是连续的 。但其实他更重要的作用是减少带宽的消耗，因为在读取贴图的时候，在正常UV0-1的连续读取中，GPU会尽量将这个shader中读取这张贴图的指令拼接在一起，GPU会将贴图中第一个读取指令的uv值位置周围一片像素读进L2、L1缓存中，这就是贴图读取的预测策略。这样做的好处是由于uv的连续，会有高命中率直接获取所需uv值的颜色值；而不连续的uv值将需要获取图片中2个距离很大的区域，比如如果当前像素线程中uv值是(0,0)，那么读取贴图的时候，GPU会将索引为(0,0)周围区域的像素一并读取进L2、L1缓存中；但如果我们右边第二个像素值却是(0.5,0.5)（这在距离摄像机远的物件中，是非常常见的顶点UV插值后的结果），这样的话，我们在读取贴图的时候当前内存中不存在这个贴图的uv位置的数据，那么我们只能清空当前已经进入缓存的数据，再重新读取贴图对应uv(0.5,0.5)周围一片的像素的值，再次放入L2、L1缓存中，这就是我们经常说的缓存命中率低造成的带宽上升的问题。
      我的理解：开启mipmap后，显卡会先计算当前像素的mipmap level，根据mipmap level去读取对应等级的贴图，让两个相邻像素的所读取的贴图位置相邻，提高Cache Hit

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T2900-17.png)

15. 而接下来的操作就和vertex shader中的逻辑步骤完全一样了，但是变成了在像素着色器线程中执行。由于不耗费任何性能就可以获取一个像素内的值，导致锁步执行非常便利，所有的线程内可以保证其指令在同一点

16. 最后一步，现在的像素着色器已经完成了颜色的计算和深度值的计算，在这个点上，我们必须考虑三角形的原始API顺序，然后才将数据移交给ROP（Render Output Unit，渲染输出单元），一个ROP内部有很多ROP单元，在ROP单元中处理深度测试和frambuffer的混合，深度和颜色的设置必须是原子操作，否则两个不同的三角形在同一个像素点就会有冲突和错误

![2.9-18](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/cd7ce2fb-d2a6-4682-9904-ecf4d818a49c.png)

## 3 Early-Z

早期的GPU的渲染管线测深度测试是在像素着色器之后才执行的，这样会造成很多根本不可见的像素执行了耗性能的像素着色器计算。后来，为了减少像素着色器的额外消耗，将深度测试提前制像素着色器之前（如下图所示），这就是Early-Z技术的由来。Early-Z技术可以将很多无效的像素提前剔除，避免他们进入耗时严重的像素着色器。Early-Z剔除的最小单位不是1像素，而是像素块（2*2）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/c6f913dd-8850-430e-ac9d-65e1f7722680.png" alt="2.9-19" style="zoom: 67%;" />

但是，以下情况会导致Early-Z失效：

1. 开启Alpha Test：由于Alpha Test需要在像素着色器后的Alpha Test阶段比较（DX的discard，OpenGL的Clip），所以无法在像素着色器之前就决定该像素是否被剔除
2. 开启Alpha Blend：启用了Alpha混合的像素很多需要与frame buffer做混合，无法执行深度测试，也就无法利用Early-Z技术。
3. 关闭深度测试：Early-Z是建立在深度测试开启的条件下，如果关闭了深度测试，也就无法启用Early-Z技术
4. 开启Muti-Sampling：多采样会影响周边像素，而Early-Z阶段无法得知周边像素是否被裁剪，故无法提前剔除
5. 以及其他任何导致需要混合后面颜色的操作

## 4 SIMD与SIMT

#### SIMD

SIMD，Simgle Instruction Mutiple Data，是单指令多数据，在GPU的ALU单元内，一条指令可以处理多维向量（一般是4D）的数据。比如，有以下shder指令：

```glsl
float4 c = a + b; //a, b都是float4类型的数据
```

对于没有SIMD的处理单元，需要4条指令将4个float数值相加，汇编代码如下：

```nasm
ADD c.x, a.x, b.x
ADD c.y, a.y, b.y
ADD c.z, a.z, b.z
ADD c.w, a.w, b.w
```

但有了SIMD技术后，只需要一条指令就可以完成，汇编代码如下：

```nasm
SIMD_ADD c,a,b
//这就相当于for(i=0;i<n;++i) a[i]=b[i]+c[i];
```

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/df086102-efb2-4621-aeb2-32ea37d65ca9.png" alt="2.9-20" style="zoom:80%;" />

#### SIMT

SIMT，Single Instruction Multiple Thread，是单指令多线程，是SIMD的升级版，可对GPU中单个SM中的多个Cire同时处理同一指令，并且每个Core存取的数据可以是不同的。

其汇编代码如下：

```nasm
SIMT_ADD c,a,b
```

上述指令会被同时送入在单个SM中被编组的所有Core中，同时执行运算，但a、b、c的值都可以不一样：

```nasm
__global__void add(float *a, float *b, float *c){
int i = blockIdx.x * blockDim.x + threadIdx.x;
a[i] = b[i] + c[i]; //no loop!
}
```

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/5cfc2c48-c718-4b3e-ab63-53a60068f59f.png" alt="2.9-21" style="zoom:80%;" />

![9aed3ff5-db7b-42e8-b4a3-59bd9609a13d.png](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/9aed3ff5-db7b-42e8-b4a3-59bd9609a13d.png)

#### co-issue

co-issue是为了解决SIMD运算单元无法充分利用的问题。例如下图，由于float数量的不同，ALU利用率从100%依次下降为75%、50%、25%

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/b6e2d7ce-fc20-4eb6-8b83-0c83f64a9c4b.png" alt="2.9-22" style="zoom: 67%;" />

为了解决着色器在低维向量的利用率低的问题，可以通过合并1D与3D，或合并2D与2D的指令。例如下图，DP3指令用了3D数据，ADD指令只有1D数据，co-issue会自动将他们合并，在同一个ALU中只需要一个指令周期即可执行完

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/d2ca8f1f-b223-42b4-9cec-f206f184b126.png" alt="2.9-23" style="zoom: 80%;" />

但是对于向量运算单元（Vetor ALU），如果其中一个变量既是操作数又是存储数的情况，无法启用co-issue技术

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/619a2a5c-4707-4fbd-bd09-bb54953a3a07.png" alt="2.9-24" style="zoom:80%;" />

## 5 CPU与GPU的对比

CPU是一个具有多种功能的优秀领导者，他的优点在于调度、管理、协调能力强，但计算能力一般

GPU相当于一个接受CPU调度的”拥有大量计算能力“的员工

![4302207a-2747-48d0-b58c-93b0703bcfc9.png](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/4302207a-2747-48d0-b58c-93b0703bcfc9.png)

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/94f04abd-2836-431a-be10-921920d9ad5a.png" alt="2.9-25" style="zoom:80%;" />

## 6 CPU-GPU的异构系统

根据CPU和GPU是否共享内存，可分为两种类型得CPU-GPU架构：Discrete分离式架构，和Coupled耦合式架构

- 分离式架构中，CPU和GPU各自有独立的缓存和内存，他们通过PCI-e等总线通讯。这种结构的缺点在于PCI-e相对于两者具有低带宽和高延迟，数据的传输成了其中的性能瓶颈。目前使用非常广泛，如PC等

  ![2.9-26](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/0006348d-8fd8-48fd-a5b9-2b6174882a0c.png)

- 耦合式架构中，CPU和GPU共享内存和缓存。AMD的APU采用的就是这种结构，目前主要用在游戏主机中，如PS4，智能手机

  ![2.9-27](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/190f0a37-8c4d-4328-b466-41da202c6fa9.png)

在内存管理方面，分离式结构中CPU和GPU各自拥有独立的内存，两者共享一套虚拟地址空间，必要时会进行内存拷贝。对于耦合式结构，GPU没有独立的内存，与CPU共享内存，由MMU(Memory Management unit)进行存储管理

## 7 GPU资源机制

#### 7.1 GPU内存架构

GPU与CPU类似，也有多级缓存结构：寄存器、L1缓存、L2缓存、GPU显存、系统内存

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/237a8163-2067-4f70-bfbc-67c3acda6135.png" alt="2.9-28" style="zoom:80%;" />

他们的存取速度从寄存器到系统内存依次变慢。由下图可见，GPU读取寄存器时，消耗一个访问周期，而共享内存L1缓存是1\~32个，L2缓存是32\~64个，而纹理、常量缓存和全局内存达到了惊人的400\~600

![4683eb74-02b1-40f7-b9b8-eb14eb72eb5e.png](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/4683eb74-02b1-40f7-b9b8-eb14eb72eb5e.png)

由此可见，shader直接访问寄存器、L1、L2缓存还是比较快的，但是访问纹理、常量缓存和全局内存会非常慢，会造成很高的延迟。这也是为什么我们要增加纹理的缓存命中率，尽量避免cache-missing

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/4aec7a55-008f-4586-b327-dffb24fff640.png" alt="2.9-29" style="zoom: 33%;" />

#### 7.2 GPU中的各个内存

GPU内存分布在RAM存储芯片或者GPU芯片上，他们物理上所在的位置决定了他们的速度、大小以及访问规则

- 全局内存（Global Memory）：位于片外存储体中，容量大，但访问延迟高、传输速度较慢，使用L2 Cache做缓存
- 本地内存（Local Memory）：一般位于片内存存储体中，变量、数组、结构体等都存放在此处，但是有大数组、大结构体以至于寄存器区放不下他们，编译器在编译阶段就会将他们放到片外的DDR芯片中（最好的情况也会被扔到L2 Cache中），且将他们标记为”Local“型
- 共享内存（Shared Memory）：位于每个流处理器组（SM）中，其访问速度仅次于寄存器
- 寄存器内存（Register Memory）：位于每个流处理器组（SM）中，访问速度最快的存储体，用于存储线程执行时所需的变量
- 常量内存（Constant Memory）：位于每个流处理器组（SM）中和片外的RAM存储器中
- 纹理内存（Texture Memory）：位于每个流处理器组（SM）中和片外的RAM存储器中

#### 7.3 GPU资源管理模型（分离式架构）

首先从图右下角可知，CPU与GPU之间的交流通过MMIO进行，我们的驱动程序通过MMIO获取寄存器的状态，也通过MMIO进行数据的传输。

图中间的GPU Context，他位于驱动程序所管理的虚拟内存当中，GPU可以并存多个活跃状态下的context，也就是多个GPU上下文。

所有的contex通过page table隔离，提交命令到硬件单元，也就是GPU Channel，每个GPU Channel会关联一个context，而一个GPU Context可以提交给多个GPU Channel。

我们的指令通过channel发送到core中的执行上下文模块存储，等待调度执行。在执行完毕后，将计算结果写回虚拟内存中，然后虚拟内存通过DMA（Direct Memory Access，一种高速的数据传输操作）传输回我们的主存。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/fce9fcca-b3d3-4e99-92fc-e85578ef761d.png" alt="2.9-30" style="zoom:80%;" />

简单来说，CPU到GPU的数据流如下所示

1. 将主存的数据复制到显存中
2. CPU通过指令驱动GPU
3. GPU中的每个运算单元并行处理（此步会在显存上存取数据）
4. GPU将现存结果传回主存

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/71b7b653-1778-488f-ad66-61a4baf9aece.png" alt="2.9-31" style="zoom:67%;" />

## 8 硬件层面下Shader的运行机制

执行阶段，CPU端会将已经在离线阶段编译好的shader汇编代码经由PCI-e推送到GPU端，GPU在执行代码时，会用Context将指令分成若干Channel推送到各个Core的存储空间中

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/004112f1-93e2-4c86-b2d4-0df5d799d374.png" alt="2.9-32" style="zoom: 80%;" />

下面就是一个假象的Core的示意图，一个GPU Core包含8个ALU，4组执行环境（Execution Context），每组执行环境有8个GPU Context。这样，一个Core可以并发(Concurrent but interleaved)执行4条指令流(Instruction Streams)，32个并发程序片元（Fragment）

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/4519013b-2907-42a8-b4cc-65b33e27c2ce.png" alt="2.9-33" style="zoom:80%;" />

以如下漫反射为例：

```glsl
sampler mySamp;
Texture2D<float3> myTex;
float3 lightDir;

float4 diffuseShader(float3 norm, float2 uv)
{
		float3 kd;
		kd = myTex.Sample(mySamp, uv);
		kd *= clamp(dot(lightDir, norm), 0.0, 1.0);
		return float4*kd, 1.0);
}
```

在执行阶段，汇编代码会被GPU推送到执行上下文（Execution Context），然后ALU会逐条获取）Detch）、解码（Decode）汇编指令为二进制指令，并执行他们。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/4ffd9380-f5e7-40ca-bdf4-0d8622e48ef8.png" alt="2.9-34" style="zoom: 67%;" />

对于SIMT架构的GPU，汇编指令有所不同，变成了SIMT特定指令代码

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/0551789e-85e9-4c68-a8b6-685eef70c801.png" alt="2.9-35" style="zoom:67%;" />

并且Context以Core为单位组成共享的结构，同一个Core的多个ALU共享一组Context；如果有多个Core，就会有更多的ALU同时参与shader计算，每个Core执行的数据是不一样的，可能是顶点、图元、像素等任何数据。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/296dbb35-9069-43cb-80c8-9031da0f7d8e.png" alt="2.9-36" style="zoom: 50%;" />

## 9 GPU Context和延迟

由于SIMT技术的引入，导致很多同一个SM内的很多Core并不是独立的，当它们当中有部分Core需要访问到纹理、常量缓存和全局内存时，就会导致非常大的卡顿（Stall）

例如下图中，有4组上下文（Context），它们共用同一组运算单元ALU。

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/9b7c05c2-3a92-4352-b20d-9b523a131e14.png" alt="2.9-37" style="zoom: 50%;" />

假设第一组Context需要访问缓存或内存，会导致2~3个周期的延迟，此时调度器会激活第二组Context以利用ALU。当第二组Context访问缓存或内存又卡住，会依次激活第三、第四组Context，直到第一组Context恢复运行或所有都被激活

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/4aa3972f-2f7c-41ee-9eba-00f6d6b6fa98.png" alt="2.9-38" style="zoom:50%;" />

延迟的后果是每组Context的总体执行时间被拉长了

越多Context可用就越可以提升运算单元的吞吐量，比如下图的18组Context的架构可以最大化地提升吞吐量：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/2e086ab5-1e2a-4bc3-918c-6b4b59256720.png" alt="2.9-39" style="zoom: 50%;" />

我们可以理解为，我们需要在shader中读取两张贴图，并且进行大量的计算。此时上下文会进行切割，尽量将不相关的计算分为不同的上下文，以便于分开执行。我们在第一组warp的32个线程指令走到读取第一个图后，它会将指针指向读取贴图的结果，然后开始切换到第二组warp中32个线程执行其余上下文。直到第二组warp又开始读取图片，这样又会激活第三组、第四组等。这样可以充分的利用我们读取图片内存卡住的一个时间

## 10 总结及渲染优化建议

- 顶点着色器和像素着色器都是在同一个单元中执行的（在原本的架构中中vs和ps的确实分开的，后来NVIDIA把这个统一了），vs是按照三角形来并行处理的，ps是按照像素来并行处理的
- vs和ps中的数据是通过L1和L2缓存传递的
- warp和thread都是逻辑上的概念，sm和sp都是物理上的概念。线程束≠流处理器数

优化建议：

1. 尽量使用自己扩展的几何实例化替代unity提供的静态合批、动态合批，静态合批将合并mesh增加vbo内存占用，动态合批则会增加CPU端的耗时开销
2. 尽量减少顶点数与三角形面数，前者减少顶点着色器的运算和显存中frameData的内存存储，后者减少片元着色器的消耗
3. 避免每帧提交buffer的数据，比如unityCPU版本的粒子系统，可使用GPU版本的粒子系统替代，将修改数据移动到GPU端；另外特别提醒的就是避免大片的透明粒子特效，这将造成严重的overdraw
4. 减少渲染状的设置和获取，例如在Updare中获取设置shader的属性或者公共变量，因为前面提到CPU是通过MMIO获取寄存器数据，这将浪费更多的时间周期
5. 3D物件硬使用LOD减少处理的顶点与面数的消耗，开启mipmap减少贴图缓存命中的丢失
6. 避免AlphaTest的使用，这会导致Early-Z的失效
7. 避免三角面过小，这会加剧overdraw的情况，也就是前面提到的一个三角形只占据3个像素点，却使用了12个线程去计算像素值，然后遮蔽其余9个的计算结果
8. 在寄存器数量与变体数量中寻找平衡，使用if变量达成静态分支，取代变体，一方面可以减少变体数量，另一方面也可以使得URP中的SRP Batch更高效地合批
9. 尽量避免判读分支，也就是shader中if true和false都会走的情况
10. 减少复杂函数的调用，因为从硬件架构上可以看出特殊函数处理单元是远远少于正常计算单元的
