---
title: 【Unity】仿原神渲染2.0技术文档-后处理篇
date: 2023-12-07 21:17:30
tags:
 - Unity
 - URP管线
 - 卡通渲染
 - SRP自定义后处理
 - 技术美术
categories: Tech-Document
description: 在URP管线下探究原神的卡通渲染实现的技术分享文档，后处理部分
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/73410871d211425732b5205f15856be.png
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/c29cb1345f98a9f92545947d81d692c.png
---

# 【Unity】URP 中的仿原神渲染2.0-后处理篇

> 过程中使用到的模型、贴图均来自miHoYo，个人仅用作学习使用

## 整体效果展示

【【卡通渲染/MMD】五等分の気持ち】 https://www.bilibili.com/video/BV1ew41187H5/

![夜间、白天及加入Bloom后的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204162907101.png)

> 代码仓库链接：https://github.com/himoqiuhan/URPToonRenderReconstruction



## 0 前言

这一次渲染的后处理我只自己写了Bloom，不过整体的后处理代码架构我个人认为是比较清晰的，也比较方便用于拓展。



## 1 Bloom处理思路

Bloom处理的核心有三步，第一步提取Bloom区域，第二步对提取结果进行模糊处理，第三步合并处理结果与原图。

![Bloom处理核心步骤](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205160707729.png)

提取Bloom区域的方式一般有两种，一是直接根据颜色值提取，二是根据亮度提取。因为两个的处理方式都很简单，所以我在处理流程中加入了枚举，可以选择不同的提取方式。不过一般情况下，直接根据颜色值来提取得到的效果要更好一点，因为颜色值减去阈值的时候是RGB值都减去一个相同的数，降低明度的同时还能够保持颜色的饱和度。

对提取结果进行模糊处理时，最简单的方法是直接进行高斯模糊，但是那样无法获得“扩散”的感觉。我的处理方案参考了COD2014年的分享，对Bloom区域提取出来的结果进行逐级降采样，存储到一系列临时RenderTexture中，然后再逐级向上叠加混合，最后对最终的混合结果进行一次高斯模糊。其中一些提升效果的细节，我放到下面详细介绍。

合并处理结果与原图就是控制Bloom强度后与原图直接相加。



## 2 添加Feature实现URP自定义后处理

### 2.1 RenderFeature与RenderPass

通常在Project视口中右键->Create->Rendering->URP Asset(with Universal Renderer)可以新建一个Render Feature模板，里面有官方给的注释，但是模板中Pass类的定义是写在RenderFeature类中的，我想要架构更清晰一点，同时想要一个Feature处理多个不同的后处理Pass，就把他们放到了不同的文件中。我没有考虑过一个Feature包含多个Pass的性能问题，如果有大佬知道，还请不吝赐教！

这样就获得了如下两个文件：CustomPostProcessFeature.cs和CustomBloomPass.cs

CustomPostProcessFeature.cs统筹管理所有后处理相关的Pass。这一个版本只写了Bloom，后续添加其他的后处理Pass同Bloom处理即可。这样处理需要在代码中考虑不同效果执行的先后顺序，以及目前有一点可能的优化空间：如果加入没有开启的Pass，是否执行Pass的逻辑是在Pass的Execute函数中判断的，也就是说该Renderer一定会执行EnqueuePass的函数。

RenderFeature的关键函数是Create和AddRenderPasses，Create用于创建对应ScriptableRenderPass的对象，并设置RenderPassEvent，即把这个Pass加入到哪个阶段，AddRenderPasses用于将Pass加入到CommandBuffer的队列中，用于后续执行。

```c#
public class CustomPostProcessFeature : ScriptableRendererFeature
{

    CustomBloomPass m_CustomBloomPass;

    public override void Create()
    {
        m_CustomBloomPass = new CustomBloomPass();

        // Configures where the render pass should be injected.
        m_CustomBloomPass.renderPassEvent = RenderPassEvent.BeforeRenderingPostProcessing;
    }

    // Here you can inject one or multiple render passes in the renderer.
    // This method is called when setting up the renderer once per-camera.
    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        m_CustomBloomPass.SetUp(renderer.cameraColorTarget);
        renderer.EnqueuePass(m_CustomBloomPass);
    }
}
```

CustomBloomPass.cs执行具体的Bloom处理指令，指令存放在Execute函数中，其他的内容基本是围绕着这个函数进行处理，代码框架如下：

```c#
public class CustomBloomPass : ScriptableRenderPass
{
    //需要使用到的成员变量，例如ShaderProperty的ID、用于执行的Shader、以及用于处理的Material等等，具体参数可以看完整的代码
    //...
    
    public CustomBloomPass()
    {
        //基于设置的Shader创建材质
        //...
    }
    
    public void SetUp(in RenderTargetIdentifier finalRenderTarget)
    {
        //SetUp函数由RenderFeature的AddRenderPasses调用，传递最终的Target信息
        _finalRenderTarget = finalRenderTarget;
    }

  public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        if (!GetParamsAndCheckValidation(ref renderingData))
        {
            // Debug.Log("Custom Bloom Failed");
            return;
        }

        var cmd = CommandBufferPool.Get(BufferName);
        Render(cmd, ref renderingData);
        context.ExecuteCommandBuffer(cmd);
        CommandBufferPool.Release(cmd);
    }
  
    public override void OnCameraCleanup(CommandBuffer cmd)
    {
        ReleaseRenderTextures(cmd);
    }

    private bool GetParamsAndCheckValidation(ref RenderingData renderingData)
    {
         //检查Material是否被成功创建、控制参数的Volume是否有效、对应后处理是否启用
	    //...
    }

    private void Render(CommandBuffer cmd, ref RenderingData renderingData)
    {
        //获取相机的相关信息，并执行渲染相关的函数
        ref var cameraData = ref renderingData.cameraData;
        RenderTargetIdentifier source = _finalRenderTarget;
        var w = cameraData.camera.scaledPixelWidth;
        var h = cameraData.camera.scaledPixelHeight;

        CreateRenderTextures(cmd, w, h);
        
        BloomAreaExtractionPassRender(cmd, source, _bloomRTArray[0]);
        BloomPassRender(cmd, source, _finalRenderTarget);
    }

    private void CreateRenderTextures(CommandBuffer cmd, int w, int h)
    {
        //基于需求创建临时RT
        //...
    }

    private void ReleaseRenderTextures(CommandBuffer cmd)
    {
        //释放所创建的临时RT
        //...
    }

    private void BloomAreaExtractionPassRender(CommandBuffer cmd, RenderTargetIdentifier source, RenderTargetIdentifier target)
    {
        //提取Bloom区域Pass的执行
        //...
    }

    private void GaussianBlurPassRender(CommandBuffer cmd, RenderTargetIdentifier source, RenderTargetIdentifier target)
    {
        //处理高斯模糊的双Pass执行
        //...
    }
    
    private void BloomPassRender(CommandBuffer cmd, RenderTargetIdentifier source, RenderTargetIdentifier target)
    {
        //对Bloom区域进行降采样、升采样与最终Bloom效果的混合
        //...
    }
}
```



### 2.2 参数控制器

参数控制器我放在了Volume中，作为后处理的一个组件，所以控制效果的参数放在了一个继承自VolumeComponent和IPostProcessCompoment的类中。这样处理的优势是，可以方便地在一个Global Volume中与URP其他的后处理结合使用，同时制作分镜所用的Cinemachine支持在Timeline的shot中使用不同的Volume Profile控制不同镜头的后处理效果。代码如下：

```c#
[Serializable, VolumeComponentMenuForRenderPipeline("Custom Post-processing/Bloom", typeof(UniversalRenderPipeline))]
public class CustomBloomParams : VolumeComponent, IPostProcessComponent
{
    //Activity Controller
    public bool IsActive() => useCustomBloom.value;
    public bool IsTileCompatible() => false;


    [Serializable]
    public sealed class BloomAreaExtractionModeParameter : VolumeParameter<BloomAreaExtractionMode> {} 
    public enum BloomAreaExtractionMode 
    {
        Luminance,
        Color
    }

    //Parameters
    public BoolParameter useCustomBloom = new BoolParameter(false);

    [Header("Bloom Perception")]
    public BloomAreaExtractionModeParameter bloomAreaExtractBy = new BloomAreaExtractionModeParameter();
    public ClampedFloatParameter bloomThreshold = new ClampedFloatParameter(0.0f, 0.0f, 1.0f);
    public ClampedFloatParameter bloomIntensity = new ClampedFloatParameter(1.0f, 0.0f, 2.0f);
    public ClampedFloatParameter bloomScatter = new ClampedFloatParameter(0.5f, 0.0f, 1.0f);
    public ColorParameter bloomColorTint = new ColorParameter(Color.white);

    [Header("Bloom Quality")] 
    public ClampedIntParameter gaussianBlurIterations = new ClampedIntParameter(1, 1, 4);
    public ClampedFloatParameter gaussianBlurSize = new ClampedFloatParameter(0.6f, 0.2f, 3.0f);
    public ClampedIntParameter basicDownSample = new ClampedIntParameter(1, 1, 4);
    public ClampedFloatParameter finalGaussianBlurBlurSizeScale = new ClampedFloatParameter(4, 1, 10);

    [Header("Bloom Details")] 
    public ClampedFloatParameter mipIntensity1 = new ClampedFloatParameter(0.75f, 0.5f, 1.0f);
    public ClampedFloatParameter mipIntensity2 = new ClampedFloatParameter(0.85f, 0.5f, 1.0f);
    public ClampedFloatParameter mipIntensity3 = new ClampedFloatParameter(0.95f, 0.5f, 1.0f);
    public ClampedFloatParameter mipIntensity4 = new ClampedFloatParameter(1.0f, 0.5f, 1.0f);
}
```

基于我的处理流程，有几个特殊的参数：BloomScatter用于控制最终表现效果“扩散”程度；BloomColorTint用于调整Bloom的颜色；MipIntensity1-4是用于控制最多4个模糊Mip混合的强度（数量由高斯模糊迭代次数决定），表现在从内到外最多四层的Bloom强度的分别控制。

![作为PostProcess组件表现在Volume中的控制器](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205170351452.png)



## 3 获取Bloom区域

开篇提到过，提取Bloom区域有两种方式，一是直接根据颜色值提取，二是根据亮度提取。通过简单的ShaderFeature的控制，就可以把两个处理方式简化到一个Pass中进行。对优化应该没有影响，但是这样做逻辑会清晰很多。所以需要处理的就是在BloomAreaExtractionPassRender中设置不同的宏，代码如下：

```c#
_material.SetFloat(BloomThresholdID, _params.bloomThreshold.value);
if (_params.bloomAreaExtractBy.value == CustomBloomParams.BloomAreaExtractionMode.Luminance)
{
    _material.EnableKeyword("_EXTRACTBYLUMINANCE");
    _material.DisableKeyword("_EXTRACTBYCOLOR");
}
else if (_params.bloomAreaExtractBy.value == CustomBloomParams.BloomAreaExtractionMode.Color)
{
    _material.EnableKeyword("_EXTRACTBYCOLOR");
    _material.DisableKeyword("_EXTRACTBYLUMINANCE");
}
else
{
    _material.DisableKeyword("_EXTRACTBYCOLOR");
    _material.DisableKeyword("_EXTRACTBYLUMINANCE");
}
```

Shader方面则是在FragmentShader中对不同的宏进行不同的处理：基于亮度提取的话，需要计算出画面的亮度，然后用亮度减去提取阈值；而基于颜色提取，则直接对RGB同时减去阈值即可，相关代码如下：

```hlsl
#if defined(_EXTRACTBYLUMINANCE)
	float luminance = 0.2126 * mainTexCol.r + 0.7152 * mainTexCol.g + 0.0722 * mainTexCol.b;
	float mask = max(luminance - _ExtractThreshold, 0.0);
	bloomAreaColor = mainTexCol * mask;
#elif defined(_EXTRACTBYCOLOR)
	bloomAreaColor = max(mainTexCol - _ExtractThreshold, 0.0);
#else
	bloomAreaColor = mainTexCol;
#endif
```

![通过亮度提取Bloom区域(左)与通过颜色提取Bloom区域(右)提取后的结果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205205509026.png)



## 4 双Pass高斯模糊

高斯模糊的算法原理已经有很多文章写得很清楚了，我不在此赘述，而高斯模糊通过双Pass来降低采样贴图的次数也是人尽皆知的优化方法了，项目中这一部分也只是把两个Pass的执行优化到了两次Blit完成，并且封装到函数中，方便多处使用。封装后处理高斯模糊的函数如下：

```c#
private void GaussianBlurPassRender(CommandBuffer cmd, RenderTargetIdentifier source, RenderTargetIdentifier target)
{
    cmd.Blit(source, GaussianBlurTempTargetID, _material, 1);
    cmd.Blit(GaussianBlurTempTargetID, target, _material, 2);
}
```

Shader代码和《UnityShader入门精要》中的代码相似，借助\_BlurSize控制卷积核大小，来保持模糊效果的同时降低迭代次数，以实现优化。具体代码可以看我的Git仓库，为了文章精简，我就不贴在这里了。



## 5 降、升采样与混合

降采样的原理时使用低分辨率的图存储上一级分辨率模糊处理后的图，将贴图的filtermode设置为Blinear，降低采样率时（目标为低分辨率RT）利用双线性插值采样来获得平滑的更模糊的图片，升高采样率时（目标为高分辨率RT）平滑低分辨率到高分辨率过渡的效果。

降采样获取模糊的RTMips时，为了加强模糊效果，并保持不同层级之间模糊效果的连续性，我是对上一层的RT作为源进行模糊处理，代码如下：

```c#
for (int i = 1; i < _params.gaussianBlurIterations.value + 1; i++)
{
    GaussianBlurPassRender(cmd, _bloomRTArray[i - 1], _bloomRTArray[i]);
}
```

![4个Mip层级的降采样](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205205854831.png)

我处理升采样的流程与COD中分享的有一点不同。因为创建Mips数组时，每一个元素在Shader中都有对应的Property "\_BloomRT_n"，所以我直接在一个Pass中进行了所有Mips层级的混合，并通过\_BloomScatter去控制层级之间的混合强度，用于调整”扩散“的强度。不过这一块的处理我暂时没有找到灵活的方式，所以基于高斯模糊迭代次数在shader中写了4个静态分支，也因此控制了最大高斯模糊迭代次数为4。其中最复杂的\_MipsCouts为4时，Mips混合的shader如下：

```hlsl
else if (_MipsCounts == 4)
{
    float4 bloomRT1 = SAMPLE_TEXTURE2D(_BloomRT_1, sampler_BloomRT_1, input.uv);
    float4 bloomRT2 = SAMPLE_TEXTURE2D(_BloomRT_2, sampler_BloomRT_2, input.uv);
    float4 bloomRT3 = SAMPLE_TEXTURE2D(_BloomRT_3, sampler_BloomRT_3, input.uv);
    float4 bloomRT4 = SAMPLE_TEXTURE2D(_BloomRT_4, sampler_BloomRT_4, input.uv);
    return lerp(
        lerp(
            lerp(bloomRT1 * _BloomMipIntensity.x, bloomRT2 * _BloomMipIntensity.y, _BloomScatter),
            bloomRT3 * _BloomMipIntensity.z, _BloomScatter),
            bloomRT4 * _BloomMipIntensity.w,
            _BloomScatter);

}
```

为了混合后不同Mips层级之间的平滑，再多加了一次高斯模糊的处理，最后再与原图像进行混合。升采样与最终混合的代码如下：

```c#
//Up-Sample
    //Mipmaps Blend
_material.SetFloat(BloomScatterID, _params.bloomScatter.value);
_material.SetFloat(Shader.PropertyToID("_MipsCounts"), _params.gaussianBlurIterations.value);
_material.SetVector(BloomMipIntensityID, new Vector4(_params.mipIntensity1.value,
    _params.mipIntensity2.value, _params.mipIntensity3.value, _params.mipIntensity4.value));
cmd.Blit(source, _bloomRTArray[0]);
cmd.Blit(source, _bloomRTArray[1], _material, 3);
    //Final Blend
_material.SetFloat(BlurSizeID, _params.gaussianBlurSize.value * _resolutionScale * _params.finalGaussianBlurBlurSizeScale.value);
GaussianBlurPassRender(cmd, _bloomRTArray[1], _bloomRTArray[1]);
_material.SetFloat(BloomIntensityID, _params.bloomIntensity.value);
_material.SetColor(BloomColorTintID, _params.bloomColorTint.value);
cmd.Blit(_bloomRTArray[0],target, _material, 4);
```

![Mips直接叠加后的结果(左)及对其进行一次高斯模糊处理后的结果(右)](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205210237925.png)

额外有一点要注意，分辨率与卷积核大小需要做对应关系的调整。在处理BlurSize时，不同的分辨率会带来不同的效果，所以为了适配分辨率的变化，需要基于分辨率去调整BlurSize，即卷积核的大小。值直接在CPU端计算一个ResolusionScale，然后直接在C#中作用到BlurSize上，不需要在GPU中重复进行计算。

```c#
private static readonly float BaseResolution = 1920f;
private float _resolutionScale;
//在初始设置的函数中计算
_resolutionScale = Math.Max(Screen.width, Screen.height) / BaseResolution;
//在设置BlurSize时对控制器中获取到的参数进行调整
_material.SetFloat(BlurSizeID, _params.gaussianBlurSize.value * _resolutionScale);
```

![Bloom开启前后的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205234805145.png)



## 6 代码架构

![GLR2后处理相关代码架构](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205234117865.png)



## 参考链接

COD在SIGGRAPH 2014上的分享，PPT第143-170页是Bloom部分：

https://www.iryoku.com/next-generation-post-processing-in-call-of-duty-advanced-warfare/

FB佬的自定义后处理教程：

https://zhuanlan.zhihu.com/p/385830661

Unity官方的自定义后处理教程：

https://zhuanlan.zhihu.com/p/373273390



## 最后

再放一遍做的MMD，虽然效果还有欠缺，但是也花了很久时间去做了。如果文章对你有所帮助的话，也请转到视频点个赞和关注，救救我惨淡的数据吧！

https://www.bilibili.com/video/BV1ew41187H5/

后续还有整个制作工作流的分享文章。
