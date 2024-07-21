---
title: 技术美术百人计划学习笔记（图形3.1 深度测试和模板测试）
date: 2023-06-20 12:01:11
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 深度测试与模板测试的基础知识以及一些案例，其中还有关于Early-Z技术的概述
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

![【图形渲染】 3.1 深度测试和模板测试](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100T3100.png)

## Stencil Test 模板测试

### 模板测试是什么

**【从管线上来理解】**

![渲染管线中的逐片元操作阶段](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909223449600.png)

渲染管线中的逐片元操作阶段

- Pixel Ownership Test：控制对当前屏幕像素的使用权限，比如PIE时只对Scene窗口或者Game窗口有使用权限，其他窗口没有使用权限
- Scissor Test：对可渲染内部部分内容的控制
- Alpha Test：透明度测试，只能实现不透明效果或全透明效果
- Stencil Test：模板测试
- Depth Test：深度测试

逐片元操作阶段是**不可编程**的，他是由渲染管线和硬件自身自我规定好的，但是我们可以对其中的内容进行配置（**高度可配置**）



**【从逻辑上来理解】**

![通过一定的条件来判断是对该片元或片元属性执行抛弃操作还是保留操作](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909223532907.png)



**【从概念上来理解】**

说到模板测试，就要先说到模板缓冲区。模板缓冲区与颜色缓冲区和深度缓冲区类似，模板缓冲区可以为屏幕上的每个像素点保存一个无符号整数值(通常的话是个8位整数)。这个值的具体意义视程序的具体应用而定。在渲染的过程中，可以用这个值**与一个预先设定的参考值相比较，根据比较的结果来决定是否更新相应的像素点的颜色值**。这个比较的过程被称为模板测试。模板测试发生在透明度测试（alpha test）之后，深度测试（depth test）之前。如果模板测试通过，则相应的像素点更新，否则不更新。

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909223611293.png)



### 语法表示

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909223626956.png)

**比较函数（Comparison Function）**

|   语义   |                           解释                           |
| :------: | :------------------------------------------------------: |
| Greater  |  相当于“>”操作，即仅当左边>右边，模板测试通过，渲染像素  |
|  GEqual  | 相当于“>=”操作，即仅当左边>=右边，模板测试通过，渲染像素 |
|   Less   |  相当于“<”操作，即仅当左边<右边，模板测试通过，渲染像素  |
|  LEqual  | 相当于“<=”操作，即仅当左边<=右边，模板测试通过，渲染像素 |
|  Equal   |  相当于“=”操作，即仅当左边=右边，模板测试通过，渲染像素  |
| NotEqual | 相当于“!=”操作，即仅当左边!=右边，模板测试通过，渲染像素 |
|  Always  |      不管公式两边为何值，模板测试总是通过，渲染像素      |
|  Never   |    不敢公式两边为何值，模板测试总是失败 ，像素被抛弃     |

**更新值**

|   语义   |                             解释                             |
| :------: | :----------------------------------------------------------: |
|   Keep   |        保留当前缓冲中的内容，即stencilBufferValue不变        |
|   Zero   |           将0写入缓冲，即stencilBufferValue值变为0           |
| Replace  | 将参考值写入缓冲，即将referenceValue赋值给stencilBufferValue |
| IncrSat  | stencilBufferValue加1，如果stencilBufferValue超过255了，那么保留为255，即不大于255 |
| DecrSat  | stencilBufferValue减1，如果stencilBufferValue超过为0，那么保留为0，即不小于0 |
|  Invert  |        将当前模板缓冲值（stencilBufferValue）按位取反        |
| IncrWrap | 当前缓冲的值加1，如果缓冲值超过255了，那么变成0，（然后继续自增） |
| DecrWrap | 当前缓冲的值减1，如果缓冲值已经为0，那么变成255，（然后继续自减） |



### 模板测试总结

- 使用模板缓冲区最重要的两个值：当前模板缓冲值（stencil Buffer Value）和模板参考值（reference Value）
- 模板测试主要是对这两个值使用特定的比较操作：Never、Always、Less、LEqual、Greater、Equal
- 模板测试之后要对模板缓冲区的值（stencil Buffer Value）进行更新操作，更新操作包括：Keep、Zero、Replace、IncrSat、DecrSat、Invert等等
- 模板测试之后可以根据结果对模板缓冲做不同的更新操作，比如模板测试成功操作Pass，模板测试失败操作Fail，深度测试失败操作ZFail，还有正对正面和背面精确更新操作PassBack，PassFront，FailBack等等



### 模板测试扩展

掌握模板测试的核心思想，利用模板测试的特性同其他测试或图形算法结合使用

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909223959882.png)



### 参考资料

[https://blog.csdn.net/u011047171/article/details/46928463](https://blog.csdn.net/u011047171/article/details/46928463)

[https://blog.csdn.net/liu_if_else/article/details/86316361](https://blog.csdn.net/liu_if_else/article/details/86316361)

[https://gwb.tencent.com/community/detail/127404](https://gwb.tencent.com/community/detail/127404)

[https://learnopengl-cn.readthedocs.io/zh/latest/04 Advanced OpenGL/02 Stencil testing/](https://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/02%20Stencil%20testing/)

[https://www.patreon.com/posts/14832618](https://www.patreon.com/posts/14832618)

[https://www.udemy.com/course/unity-shaders/](https://www.udemy.com/course/unity-shaders/)



## Depth Test 深度测试

深度测试的一些效果：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224038116.png)

### 深度测试的认识

**【从渲染管线上来理解】**

在逐片元操作之中，模板测试之后，透明度混合之前进行的，同样是高度可配置的

其中在片元着色器之前还有一个Early-Z技术，利用了与深度测试相同的技术

**【从逻辑上理解】**

```glsl
if(Zwrite On && (correntDepthValue ComparisonFunction DepthBufferValue)){
		写入深度
}else{
		忽略深度
}
```

```glsl
if(correntDepthValue ComparisonFunction DepthBufferValue){
		写入深度缓冲
}else{
		不写入深度缓冲
}
```

**【从概念上理解】**

所谓深度测试，就是针对当前对象在屏幕上（更准确的说是frame buffer）对应的像素点，将对象自身的深度值与当前该像素点缓存的深度值进行比较，如果通过了，本对象在该像素点才会将颜色写入颜色缓冲区，否则否则不会写入颜色缓冲

**【深度测试的发展】**

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224329534.png)

### 深度缓冲区

深度缓冲就像颜色缓冲（储存所有的片段颜色：视觉输出）一样，在每个片段中储存了信息，并且（通常）和颜色缓冲有着一样的宽度和高度。深度缓冲是由窗口系统自动创建的，它会以16、24或32位float的形式储存它的深度值。在大部分的系统中，深度缓冲的精度都是24位的
**z-buffer中存储的是当前的深度信息，对于每个像素存储一个深度值**

> 通过Zwrite和ZTest来调用Z-Buffer，实现想要的渲染结果

### ZWrite

深度写入包括两种状态：ZWrite On与ZWrite Off

当我们开启深度写入的时候，物体被渲染时针对物体在屏幕（更准确地说是frame buffer）上每个像素的深度都写入到深度缓冲区；反之，如果是ZWrite Off，那么物体的深度就不会写入深度缓冲区。

但是，物体是否会写入深度，除了ZWrite这个状态之外，更重要的是需要深度测试通过，也就是ZTest通过，如果ZTest都没通过，那么也就不会写入深度了

ZTest分为通过和不通过两种情况，ZWrite分为开启和关闭两种情况的话，一共就是四种情况：

1.深度测试通过，深度写入开启：写入深度缓冲区，写入颜色缓冲区；

2.深度测试通过，深度写入关闭：不写深度缓冲区，写入颜色缓冲区；

3.深度测试失败，深度写入开启：不写深度缓冲区，不写颜色缓冲区；

4.深度测试失败，深度写入关闭：不写深度缓冲区，不写颜色缓冲区；

### ZTest

| ZTest 状态    | 描述                       |
| ------------- | -------------------------- |
| Greater       | 深度大于当前缓存则通过     |
| LEqual        | 深度小于等于当前缓存则通过 |
| Less          | 深度小于当前缓存则通过     |
| GEqual        | 深度大于等于当前缓存则通过 |
| Equal         | 深度等于当前缓存则通过     |
| NotEqual      | 深度不等于当前缓存则通过   |
| Always（Off） | 深度不论如何都通过         |
| Never         | 深度不论如何都不通过       |

> **默认ZWrite On**和**ZTest Lequal**，深度缓存一开始为无穷大

### 渲染队列

Unity中内置的渲染队列，按照渲染顺序，从先往后进行排序，**队列数越小的越先渲染，队列数越大的越后渲染**

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224506214.png)

在Unity中设置渲染队列：`Tags { “Queue” = “Transparent”}`，默认是Geometry

- 不透明物体的渲染顺序：从前往后
- 透明物体的渲染顺序：从后往前（OverDraw）

> 可以在shader的Inspector面板查看相关属性

### Early-Z技术

传统的渲染管线中，ZTest其实是在Blending阶段，这时候进行深度测试，所有对象的像素着色器都会计算一遍，没有什么性能提升，仅仅是为了得出正确的遮挡结果，会造成大量的无用计算，因为每个像素点上肯定重叠了很多计算。因此 **现代GPU中运用了Early-Z的技术，在Vertex阶段和Fragment阶段之间（光栅化之后，fragment之前）进行一次深度测试，如果深度测试失败，就不必进行fragment阶段的计算了，因此在性能上会有很大的提升。但是最终的ZTest仍然需要进行，以保证最终的遮挡关系结果正确。**

前面的一次主要是Z-Cull为了裁剪以达到优化的目的，后一次主要是Z-Check，为了检查，如下图：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224540898.png)

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224548623.png)

### 深度值

![模型空间变换](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224615387.png)

- 模型空间中：无深度信息存储
- 世界空间：存储在Z分量上
- 视察空间：存储在Z分量上（线性深度）
- 裁剪空间：准备投影，深度缓冲中存储z/w（透视投影非线性）

> 为什么深度缓冲区存储的是非线性深度而不是线性深度呢？

在精度有限的深度缓冲区中合理分配存储到精度，较近区域的深度信息分配更高的精度，较远区域的深度信息分配更低的精度（和gamma映射异曲同工）

正确的投影特性的非线性深度方程是和1/Z成正比的，这样基本做的是在Z很近的时候高精度和Z很远的时候低精度。例如1.0和2.0之间的z值，将变为1.0到0.5之间，这样在z很小的时候给了我们很高的精度，而50.0和100.0之间的z值将只占2%的浮点数的精度，这正是我们想要的。

![线性深度](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224639055.png)

![非线性深度](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224650048.png)

> 深度值的精度不足可能会带来深度冲突的问题

所以在深度缓冲区中使用非线性深度，在近处用更高精度的数据存储，可以有效减少近处的深度冲突；而远处的因为不容易被看见，即便发生深度冲突也不会造成太多影响

> 如何防止深度冲突

1. 一个技巧：让物体之间不要离得太近
2. 另一个技巧：尽可能把近平面设置得远一些，可以有效在椎体中提高精度
3. 另一个技巧：牺牲性能，使用更高精度的深度值（24→32位）

### 案例

- 先对渲染队列进行排列，再在**同一个渲染队列中按深度**进行排序
- 对于多pass shader，unity会从所以pass的所有的queue中挑选最靠前的，然后把这个物体放到一个队列中，然后他会根据pass的顺序逐pass执行

### 深度测试总结

- 使用深度缓冲区最重要的两个值：当前深度缓冲值ZBufferValue和深度参考值referenceValue，并通过比较操作获取理想的渲染效果
- Unity中的渲染顺序：先渲染不透明物体，顺序是从前往后；再渲染透明物体，顺序是从后到前
- 通过ZWrite和ZTest组合使用控制半透明物体的渲染
- 引入Early-Z技术后的深度测试相关渲染流程
- 深度缓冲区中存储的深度值为0到1范围的浮点值，且为非线性

### 深度测试扩展

![深入了解深度值在不同阶段的变化过程，通过深度测试、深度写入、深度图等方法控制深度缓冲区](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230909224809174.png)

### 参考资料

[https://blog.csdn.net/puppet_master/article/details/53900568](https://blog.csdn.net/puppet_master/article/details/53900568)

[https://learnopengl-cn.readthedocs.io/zh/latest/04 Advanced OpenGL/01 Depth testing/](https://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/01%20Depth%20testing/)

[https://docs.unity3d.com/cn/2018.4/Manual/SL-CullAndDepth.html](https://docs.unity3d.com/cn/2018.4/Manual/SL-CullAndDepth.html)

[https://blog.csdn.net/yangxuan0261/article/details/79725466](https://blog.csdn.net/yangxuan0261/article/details/79725466)

《Unity ShaderLab 开发实战详解》

《Unity Shader 入门精要》

[https://roystan.net/articles/toon-water.html](https://roystan.net/articles/toon-water.html)
