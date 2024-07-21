---
title: 【Unity】仿原神渲染2.0技术文档-角色渲染篇
date: 2023-12-07 21:17:37
tags:
 - Unity
 - URP管线
 - 卡通渲染
 - 卡通角色渲染
 - 技术美术
categories: Tech-Document
description: 在URP管线下探究原神的卡通渲染实现的技术分享文档，角色渲染部分
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/73410871d211425732b5205f15856be.png
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/c29cb1345f98a9f92545947d81d692c.png

---

# 【Unity】URP 中的仿原神渲染2.0-角色篇

## 整体效果展示

【【卡通渲染/MMD】五等分の気持ち】 https://www.bilibili.com/video/BV1ew41187H5/

![夜间、白天及加入Bloom后的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204162907101.png)

![](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GLR2_Character_GeneralShow.gif)

> 代码仓库链接：https://github.com/himoqiuhan/URPToonRenderReconstruction

## 0 前言

对于卡通渲染，尤其是赛璐璐风格的卡通渲染的简介和精髓，其实很多大佬都已经写明了，所以我不进行系统的概述。

每个人心中都有属于自己的卡通渲染，之前和其他同学交流的时候，我对卡通角色渲染的浅薄理解是，好看的卡通角色效果，两分靠建模、五分靠贴图、三分靠渲染，并且需要整个流程中所有人员的相互协调处理。这只是个人的主观认知，其中的每一步都十分重要，都要融入设计师自己的心意，我这样说只是想要进一步强调贴图对于一个角色的重要程度。除去面部的特殊处理外，其他区域只是用简单的二分光照所获得的效果如下：

![只使用BaseColor和二分阴影得到的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202192854842.png)

整体的效果还是很好看的，在此基础上，好的渲染能让角色锦上添花，接下来我就一步一步分析一下我角色渲染的处理思路。在这里要感谢一下其他大佬复刻后的无私分享，文末有我学习参考的大佬们的文章。

以下流程中我就以心海为例进行我还原过程中的处理思路的分享。



## 1 漫反射

<img src="https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204163137322.png" alt="处理完基础漫反射后的效果" style="zoom: 67%;" />

### 1.1 二分阴影和原神的Ramp

说到赛璐璐风格的卡通渲染，最容易让人想到的就是大色块、二分阴影和描边

![为什么会出现千咲？因为刚看完来自风平浪静的明天，以及很喜欢千咲，夹带私货！](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202200129315.png)

通常情况下，最直接的二分阴影可以通过step处理NdotL得到，而原神中则是通过**Ramp贴图**来实现的二分效果。基于我之前学习的知识，Ramp一开始是用于高效模拟皮肤SSS效果的（如果说错了还请大家指正），不过随着使用德越来越多，我认为Ramp图可以理解为自定义一系列的ColorTint，以光照计算得到的数据作为采样贴图的U轴坐标，采样不同光照情况下的调色颜色，用以混合BaseColor，得到最终的光影表现，通过Ramp图可以**实现对光影颜色更便利、更美术可控的控制**。基于此，原神通过对Ramp图的二分处理，一方面实现了可由美术自定义颜色的明暗区域二分，另一方面，因为二分效果需要像素之间明显的色阶变化，所以可以用很低分辨率的图片去存储Ramp信息，横向只有256个像素，多少也是省了一些存储和内存

![心海身体部分所用到的Ramp图的明暗区分部分](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202202447174.png)

从上面的截图可以看出，相隔不远的区域之间有明显的色阶变化，基于此就可以实现如下这样更具风格化的明暗区域二分效果，同时我也很喜欢明暗区域二分区域上那条暖色的分界线。

![插画风格的明暗交界处](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202203900274.png)



### 1.2 原神ilm贴图GA通道和Ramp的采样

通常情况我们看到的Ramp图都是一条的，但是为了更加贴近插画的风格，以及更高的美术可控性，原神对Ramp图进行了进一步的升级，实现身体和头发各自**五个区域**、白天和黑夜**两个状态**下更丰富的光影细节效果，所以在上面Ramp的截图中，可以发现原神的一张Ramp图内实际存有10个有关光影变化的颜色信息。

为了**区分不同光影效果的区域**（即不同Ramp作用的区域、原神Ramp贴图采样的V轴信息），我们需要配合使用一张带有区域划分信息的贴图，通过贴图中的灰度值，来偏移采样RampTexture时的V轴坐标。原神把这个灰度信息整合到了ilmTexture的A通道，ilm这个说法最早源自GGX，不过原神中似乎命名为lightmap，为了区分BakeLight的lightmap，我在本文中暂且使用ilmTexture来称呼。

![心海ilmTexture的A通道](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202210110109.png)

可以看到，ilmTexture的A通道存储了不同的灰度值，这个值便是用于区分不同区域的信息。但是，在我的平台使用时，对V轴信息的处理有一些特殊，Ramp无法正常出效果的问题困扰了我很久，后来也是一遍一遍地摸索，发现了问题所在：首先，原神的Ramp采样是基于左上角为(0，0)的，原因猜测是网上拿到的Ramp资源是基于PC截帧获取的，DX平台默认的贴图是以左上角为UV空间的原点，而Unity中默认的UV空间原点是左下角，所以需要在Y轴上做翻转；其次，有些角色其实并没有用满10个Ramp区域，会出现如同上方截图那样的留白区域，以及ilmTexture.a存储的A通道的信息也存在一些特殊性，所以基于ilm贴图的A通道和Ramp图的适配性，以及结合游戏内的表现，我加入了一个_RampCount来对ilm贴图A通道读取Ramp列序号的信息进行缩放，使得ilmTexture的A通道为0和1的区域分别是有效Ramp的最上方和最下方，不会采样到纯白的无效区域（像我这样的处理方式或许也还原得不对，但是他似乎刚刚好让我能成功的用到五个角色上面，以及如果要我自己去做Ramp，我还是会更清晰地用更准确的灰度去控制区域，不需要加入缩放的处理）

相比于此，**白天和黑夜两个状态**的处理就简单了很多，通过一个toggle控制使用白天或黑夜的状态，加上如上处理后的V轴信息，然后再映射回0-1区间即可。代码如下：_UseCoolShadowColorOrTex即为toggle对应的参数，因为toggle获得的值直接是0或1，所以两个值乘0.5即可映射回0-1；同时减去一个0.001防止采样到像素的边缘。

```hlsl
rampUV.y = 1.0 - saturate(ilmTexCol.a) / _RampCount * 0.5 + _UseCoolShadowColorOrTex * 0.5 - 0.001;
```

前文也提到，Ramp贴图的U轴是光照计算得到的数据。基于对Ramp图的观察可以发现，明暗区分的分界线是比较靠Ramp图右侧的，而贴图UV坐标是再0-1之间的，所以基于此我们可以用halfLambert来作为U轴的坐标来对Ramp进行采样。

同时，插画中为了增加角色的表现效果，会加入主观的阴影，一方面能处理衣服布料相互遮挡的关系，一方面可以塑造诸如衣服褶皱这样的细节，使得角色表现更加生动。对阴影的笔触感处理（为了叙述方便，以下均用“固有阴影”来描述此类型的阴影），我们同样可以借助遮罩来实现，因为也是单个通道就能存储的灰度信息，原神将其存储在了ilmTexture的G通道中。对此信息通过step处理之后，再乘上我们计算出的halfLambert信息，就可以获取到最基础的带有固有阴影的光照信息。

![心海ilmTexture的G通道与处理后的光照信息](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202215851888.png)

随后，再加入一个参数对光照区域进行调整，就可以得到Ramp图的U轴坐标，代码如下：其中\_LightArea是用于调整光照区域范围的参数，调整后的信息与0.99取最小值，避免采样到像素边缘。

```hlsl
float halfLambert = 0.5 * dot(mainLightDirection, input.normalWS) + 0.5;
float AOMask = step(0.02, ilmTexCol.g);
float brightAreaMask = AOMask * halfLambert;

rampUV.x = min(0.99, smoothstep(0.001, 1.0 - _LightArea, brightAreaMask));
```

最终再乘上BaseColor，获取Ramp的运用上去后的效果。

![采样Ramp图及Ramp图对BaseColor的作用效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202221911368.png)



### 1.3 进一步优化diffuse表现

因为只使用Ramp图处理的固有阴影颜色不够明显，所以我再额外通过一个参数来进一步进行调色，代码如下：其中区分\_DarkShadowColor和\_CoolDarkShadowColor来分别处理白天或夜晚的两种情况

```hlsl
half3 darkShadowColor = lerp(_DarkShadowColor.rgb, _CoolDarkShadowColor.rgb, _UseCoolShadowColorOrTex) * rampTexCol;
```

![加深处理固有阴影后的表现效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231202233941445.png)

在写代码时，想要进一步拓展可控性（源自于前几个版本对Ramp图的处理方法），我额外加入了一个ShaderFeature来额外添加一个“使用自定义亮部调色颜色”的选项，如果使用自定义的亮部调色颜色，则可以获取到额外的二分Ramp条，其调色颜色就是不启用此feature时的亮部颜色。不过由于这一版对Ramp的处理算是比较适宜，所以在最终渲染效果上，我并没有自定义亮部颜色。不过处理的思路还是保留下来。

其实也就是进一步处理rampTexCol，通过一个参数影响最接近亮部的那条Ramp的宽度，基于此进一步处理brightAreaMask，来获取亮部区域（非Ramp区）的遮罩，然后对rampTexCol进行进一步的插值处理即可，代码如下：其中\_ShadowRampWidth用来控制最后一个“Ramp条”的宽度，乘上0.1则是在Shader内对Ramp长度进行一定范围控制，\_LightAreaColorTint为自定义的亮部调色颜色

```hlsl
#if !defined(_USERAMPLIGHTAREACOLOR_ON)
brightAreaMask = step(1.0 - _LightArea, brightAreaMask + (1.0 - _ShadowRampWidth) * 0.1);
rampTexCol = lerp(rampTexCol, _LightAreaColorTint, brightAreaMask);
#endif
```

![自定义亮部颜色情况下对Ramp宽度的调节](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GLR2_ShadowRampWidthAdjust.gif)

由于原神角色建模时脸和脖子是分开的，脖子属于身体部分，以及脖子区域计算出的光照模型通常怪怪的，所以游戏中这一部分是不参与光照计算的。为了区分出不进行光照计算的区域，我在模型顶点色的B通道中绘制了一张遮罩，区分不进行光照计算的区域，并且为了与面部结合得更融洽，还额外加入了一个参数去控制这一部分的阴影颜色，并在此之后进行最终的diffuse颜色融合，其代码如下：其中\_NeckColor是脖子区域的调色颜色。

```hlsl
ShadowColorTint = lerp(_NeckColor.rgb, ShadowColorTint, input.vertexColor.b);
half3 diffuseColor = ShadowColorTint * mainTexCol.rgb;
```

![模型顶点色B通道信息，以及最终得到的diffuse效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231203001948366.png)



### 1.4原神Diffuse贴图的A通道

原神部分角色身体和头发的漫反射贴图不仅仅只存储BaseColor，其A通道也存有额外信息。我在这次处理流程中，基于A通道存储的不同的信息，将漫反射贴图分为了四类：无额外信息、常亮自发光、闪烁自发光、AlphaTest信息。通常包含神之眼的贴图都是闪烁自发光，特殊一点的像神里皮肤帽子上花瓣的贴图是AlphaTest遮罩，其余的通常进游戏看一下就能区分。面部漫反射贴图A通道存储的是腮红区域的遮罩。

不同类型贴图的区分通过一个自定义枚举去控制宏，对于自发光的处理，我的方式是统一加入一个自发光系数，通过控控制该系数来实现对应效果，如果是非自发光或AlphaTest，直接设置自发光系数为0，然后再基于自发光系数在最终得到的颜色上叠加自发光。而对于AlphaTest，则是在对应的宏定义情况加入一个Clip来实现，代码如下：其中EmissionScale是用于控制自发光强度的参数。

```shaderlab
//Properties中
[KeywordEnum(None,Flicker,Emission,AlphaTest)]_MainTexAlphaUse("Diffuse Texture Alpha Use", float) = 0.0
//...
//Pass中
#pragma shader_feature_local _ _MAINTEXALPHAUSE_NONE _MAINTEXALPHAUSE_FLICKER _MAINTEXALPHAUSE_EMISSION _MAINTEXALPHAUSE_ALPHATEST

```

```hlsl
float emissionFactor = 1.0;
#if defined(_MAINTEXALPHAUSE_EMISSION)
	emissionFactor = _EmissionScaler * mainTexCol.a;
#elif defined(_MAINTEXALPHAUSE_FLICKER)
	emissionFactor = _EmissionScaler * mainTexCol.a * (0.5 * sin(_Time.y) + 0.5);
#elif defined(_MAINTEXALPHAUSE_ALPHATEST)
	clip(mainTexCol.a - _MainTexCutOff);
	emissionFactor = 0;
#else
	emissionFactor = 0;
#endif
//...
color *= 1 + emissionFactor;
```

心海的神之眼放在身体贴图中，身体贴图A通道存储的是闪烁自发光遮罩，头发贴图的A通道则是存储了自发光强度，

![心海Diffuse贴图的A通道，以及自发光强度为0和10时的表现](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231203140709167.png)

像神里帽子上的花朵这一部分，为了在低顶点数的面片上实现平滑的花瓣效果，在Fragment Shader中进行了Clip

![神里帽子上花瓣Diffuse贴图的A通道，以及Clip的处理](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231203141054240.png)



## 2 高光反射

<img src="E:/Document/Notes/Render/【Unity】GLR2.0-角色篇.assets/image-20231204163239958.png" alt="加入高光反射的效果" style="zoom:67%;" />

### 2.1 赛璐璐卡渲高光概述

卡通角色的高光最特殊的点在于，对简化后头发天使轮高光的处理，一般可以通过各向异性来处理，但是为了更加便捷地控制，我们可以在贴图和遮罩上下功夫。其他部位的高光，因为赛璐璐风格的大色块上色特点与形状的高度概括性，可以简单地用Blinn-Phong高光模型来处理。同样也可以使用次世代卡通渲染的处理思路，为一些需要强调高光表现的区域处理遮罩和法线贴图，结合使用风格化PBR及Blinn-Phong。基于对插画笔触感的模拟，也有如使用MatCap处理镜面高光的hack。

<img src="https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231203162159734.png" alt="卡通角色头发上高度概括的天使轮高光，为什么是赛马娘？因为我太爱帝王小姐了！" style="zoom: 45%;" />

原神中，对高光的处理方式有三种，第一是普通的Blinn-Phong高光，第二是基于MatCap模拟笔触感的镜面高光，第三是须弥版本后陆续引入的次世代卡通角色渲染高光。

![MatCap贴图及神里皮肤的法线贴图](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231203162722448.png)

这一个版本的还原中，我没有使用法线贴图，只是处理BLinn-Phong高光和特定区域的MatCap高光，然后对其叠加，获取最终的高光效果。



### 2.2 ilm贴图R通道和Blinn-Phong高光

由于身体上不同物件、衣服布料需要表现不同的材质质感，以及整体统一都表现高光会让渲染效果显得很“油腻”，所以我们需要对高光的区域、以及对应区域内高光的强度进行一定的控制，借此形成对比来加强角色的表现效果。控制方式同样是使用灰度遮罩，基于灰度缩放高光的强度。

原神把控制高光强度缩放的灰度信息放在了ilmTexture的R通道中，计算出Blinn-Phong高光然后再乘上ilmTexture.a的值即可。我在处理的过程中区分了金属区域和非金属区域，通过不同的Shiness值去分别控制不同的Blinn-Phong幂的指数，区分不同的高光效果。不过对于最终的效果，非金属区域的Blinn-Phong高光我只用在了头发上，并且为了风格化，使用step进行了二分，模拟出了一个头上高光的小圆点。代码如下：

```hlsl
//Blinn-Phong计算
float3 viewDirectionWS = normalize(_WorldSpaceCameraPos.xyz - input.positionWS.xyz);
float3 halfDirectionWS = normalize(viewDirectionWS + mainLightDirection);
float hdotl = max(dot(halfDirectionWS, input.normalWS.xyz), 0.0);
//非金属高光
float nonMetalSpecular = step(1.0 - 0.5 * _NonMetalSpecArea, pow(hdotl, _Shininess)) * ilmTexCol.r;
//金属高光（MatCap高光）
float metalBlinnPhongSpecular = pow(hdotl, _MTShininess) * ilmTexCol.r;
```



### 2.3 ilm贴图B通道和MatCap高光

我不能确切地说像原神这样实现镜面高光的贴图是MatCap图，但是大家都这么称呼，我在文中也统一这样称呼。MatCap基本概念是用于做映射的，将光源、材质信息烘焙到球上，然后存到贴图中，渲染的时候直接使用ViewSpace的法线信息去采样。我真正理解原神MatCap使用方法，源自之前和同学讨论Blender渲染MMD的ToonShader中对toon这张Texture的使用。他基于屏幕空间法线信息采样Toon贴图，制作出会随着镜头上下移动变化，并且具有笔触感的下部阴影效果（模拟光从上方打下来，所以底部是阴影）。然后我顿悟了，MatCap也是基于同样的原理，基于屏幕空间法线去读取这张贴图，在屏幕空间不同朝向的面的高光表现不同，获得具有笔触感的镜面高光表现。

![一张MMD中使用的Toon贴图和原神中模拟镜面高光的MatCap贴图](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231203211639492.png)

明白原理之后，代码就很简单了。我处理流程中使用的是HClipSpcace的法线（与VS是等效的），并且在用于读取贴图前需要将信息映射到0-1之间，不过这个映射的过程也需要一点特殊处理。可以发现，这张MatCap是上下左右对称的，而我们用于去采样贴图的屏幕空间法线的xy信息都在-1到1之间，恰好可以通过中心点(0.5, 0.5)来映射UV到0-1之间。其本源的原因我暂时还没有想到，有知道的大佬希望能在评论区说一说。这样处理后实现的效果是，越是朝向摄像机的面，高光强度越高。之后用MatCap贴图读取到的信息作为Blinn-Phong高光的强度，乘上之前计算出的高光，即可获得带有笔触感、二分感的高光。代码如下：其中_MTMapBrightness适用于控制MatCap贴图的作用强度

```hlsl
float2 normalCS = TransformWorldToHClipDir(input.normalWS.xyz).xy * 0.5 + float2(0.5, 0.5);
float metalTexCol = saturate(SAMPLE_TEXTURE2D(_MetalTex, sampler_MetalTex, normalCS)).r * _MTMapBrightness;
float metalBlinnPhongSpecular = pow(hdotl, _MTShininess) * ilmTexCol.r;
float metalSpecular = ilmTexCol.b * metalBlinnPhongSpecular * metalTexCol;
```

![MatCap加入后的高光表现，心海胸口蝴蝶结部分较明显](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GLR2_Kokomi_MatCapSpec.gif)

最终对两个计算出的高光进行相加，并乘上最终的Diffuse颜色，即可获得高光颜色，代码如下：其中_MTSpecularScale用于控制MatCap高光的整体强度，\_SpecMulti用于控制所有高光的整体强度

```hlsl
half3 specularColor = (metalSpecular * _MTSpecularScale + nonMetalSpecular) * diffuseColor;
half3 color = specularColor * _SpecMulti + diffuseColor;
```

![高光强度，只计算漫反射及加入高光后的表现](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204161131870.png)



## 3 边缘光

边缘光源自于逆光，用于勾勒出对象的轮廓。因为赛璐璐风格使用大色块铺色，容易使得主体对象与背景区分得不够清楚，尤其是在灯光不强的区域。边缘光的使用可以加大主体对象与背景的区分，同时我认为边缘光可以让卡通角色显得更立体，像原神这样带有二分感的边缘光可以让角色更有“二次元味”。

<img src="https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204164400856.png" alt="Cygames赛马娘Winning Live制作技术分享，使用边缘光区分次要马娘与背景" style="zoom:67%;" />

目前效果中，边缘光使用的是屏幕空间等距边缘光，在适宜的边缘光宽度范围内得到的效果是很不错的，缺点是需要提前渲染、发送、读取深度图。屏幕空间等距边缘光的原理是，对模型沿法线进行一定程度的外扩，对比外扩前后像素的深度差距，如果像素深度差距达到一定值，则判定为边缘光区域，做进一步的边缘光处理。所以，控制边缘光的宽度本质是控制模型外扩的距离。在此基础上，我借用了NiloToon对描边调整的方式，加入了边缘光随距离的消散，代码如下：

```hlsl
//获取Fragment原本的深度值
float2 originPositionSS = positionCS_XYZ.xy / _ScaledScreenParams.xy;
float originDepth = SAMPLE_TEXTURE2D(_CameraDepthTexture, sampler_CameraDepthTexture, originPositionSS).r;
originDepth = LinearEyeDepth(originDepth, _ZBufferParams);
//基于摄像机距离调整外扩距离，进而控制边缘光宽度
float distanceFixMultiplier = GetRimLightDistanceFixMultiplier(positionCS_XYZ.z);
//获取Fragment外扩后的深度值
float2 offsetPositionSS = originPositionSS + normalCS_XY * rimLightWidth * rimLightCorrection * distanceFixMultiplier;
float offsetDepth = SAMPLE_TEXTURE2D(_CameraDepthTexture, sampler_CameraDepthTexture, offsetPositionSS).r;
offsetDepth = LinearEyeDepth(offsetDepth, _ZBufferParams);
float depthDiff = offsetDepth - originDepth;
```

其中GetRimLightDistanceFixMultiplier函数获取基于摄像机距离控制边缘光消散的参数，代码如下：因为边缘光不同于描边，边缘光的宽度更适合基于模型大小进行控制，所以我没有像处理描边那样通过摄像机距离来缩放宽度。

```hlsl
float ApplyRimLightDistanceFadeOut(float distance)
{
    return smoothstep(0.0, 0.05, distance);
}

float GetRimLightDistanceFixMultiplier(float positionVS_Z)
{
    float distanceFix;
    distanceFix = ApplyRimLightDistanceFadeOut(positionVS_Z);

    return distanceFix;
}
```

然后进一步处理深度差，获取边缘光的遮罩。我想要通过不同深度差获取不同的边缘光强度，所以没有简单地通过step去处理深度差。但是如果直接用depthDiff去进行最终边缘光计算的插值，得到的效果不够好，不同区域之间深度diff差异太大，所以对depthDiff进一步处理，获得比较平滑的Rim Light效果

<img src="https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204190004974.png" alt="不同深度差获得不同的边缘光强度" style="zoom:80%;" />

其中的参数是我调试后认为最终效果不错的值，处理后得到的结果是减小了大Diff值与小Diff值之间的差距。然后基于diffuseColor乘上一定强度获得边缘光的颜色，并与之前出来的结果进行融合，代码如下：

```hlsl
float rimLightMask = 0.75 * (saturate(rimLightDepthDiff) * 0.5 + 0.15 * step(0.1, rimLightDepthDiff));
color = lerp(color, diffuseColor * (1.0 + _RimLightStrength * 2.0), rimLightMask);

```

![边缘光遮罩，漫反射+高光及加入边缘光的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204190657917.png)



## 4 面部阴影

卡通面部阴影目前主流实现方案有两个：一个是特殊处理面部法线，因为卡通角色面部阴影的问题来自于面部图元法线朝向无法满足赛璐璐高度概括阴影的计算，所以可以通过修正模型法线来实现，也可以通过计算一个球体的光照信息映射到面部；另一个是使用SDF面部阴影贴图，相比于特殊处理法线，这个方法更易于控制、细化面部阴影的表现效果，例如概括性的伦勃朗光、以及正面打光情况下用于提升面部立体感的鼻翼侧边的阴影。原神中的面部二分阴影是借助SDF面部阴影贴图来实现的。

SDF是有向距离场的简称，关于SDF面部阴影的原理，网上有很多大佬都有分享，在此我只说一说我自己的理解。SDF面部阴影贴图实质上只是阴影遮罩贴图，以灰度信息作为阈值，去区分光照计算出来的亮面和暗面，距离场的技术也只是在合并遮罩图时用到过。先通过制作某几个特定光照角度下面部的阴影遮罩，然后计算出每个遮罩的有向距离场，最后合并这几张距离场贴图，使得阴影过度变得连续。

所以处理的重点就在于，如何去获得光照的角度信息，可以通过点乘的值来替代。在目前的项目中，我通过获取光源在xz平面投影的向量，与模型向前的向量，计算他们的点乘，用于作为面部阴影计算的光照信息。不过在我的流程中，我将SDF贴图的信息反过来作为了光照的信息，用计算出的点乘作为阈值去进行step，这样获取到的鼻翼亮光和阴影才更为正确。还有一个特殊的地方，一般SDF面部阴影图只记录一侧的光照情况，所以需要我们获取模型在世界空间中左右方向上的一条向量，来判断光源从那一侧照过来，进一步处理贴图的翻转。处理的代码如下：

```hlsl
//获取向量信息
//unity_objectToWorld可以获取世界空间的方向信息，构成如下：
// unity_ObjectToWorld = ( right, back, left)
float3 rightDirectionWS = unity_ObjectToWorld._11_21_31;
float3 backDirectionWS = unity_ObjectToWorld._13_23_33;
float rdotl = dot(normalize(mainLightDirection.xz), normalize(rightDirectionWS.xz));
float fdotl = dot(normalize(mainLightDirection.xz), normalize(backDirectionWS.xz));

//SDF面部阴影
//将ilmTexture看作光源的数值，那么原UV采样得到的图片是光从角色左侧打过来的效果，且越往中间，所需要的亮度越低。lightThreshold作为点亮区域所需的光源强度
float2 ilmTextureUV = rdotl < 0.0 ? input.uv : float2(1.0 - input.uv.x, input.uv.y);
float lightThreshold = 0.5 * (1.0 - fdotl);

half4 ilmTexCol = SAMPLE_TEXTURE2D(_ilmTex, sampler_ilmTex, ilmTextureUV);
float lightStrength = ilmTexCol.r;
```

像眼睛、眉毛这些区域，计算光照后的效果不好，所以将其设置为不计算光照的区域，在最终合并光影时将其视为阴影区域。设置遮罩的方法可以通过贴图，也可以通过顶点颜色。边缘光的叠加与身体的处理方式相同，我在这里就不重复叙述了。最终进行光影效果合并，代码如下：其中做了基于白天、夜晚不同情况下的阴影和亮部颜色调整

```hlsl
float brightAreaMask = step(lightThreshold, lightStrength);
half3 brightAreaColor = mainTexCol.rgb * _LightAreaColorTint.rgb;
half3 shadowAreaColor = mainTexCol.rgb * lerp(_DarkShadowColor.rgb, _CoolDarkShadowColor.rgb, _UseCoolShadowColorOrTex);

half3 lightingColor = lerp(shadowAreaColor, brightAreaColor, brightAreaMask) * _MainTexColoring.rgb;
half3 color = lerp(mainTexCol.rgb, lightingColor, metalTexCol.r);
```

![SDF阴影直接采样及受光照区域遮罩](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204204938806.png)

![面部的BrightAreaMask](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/GLR2_Kokomi_FaceShadow.gif)

![BrightAreaMask，只是用漫反射贴图及最终运用阴影和颜色调整后的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204204328212.png)



## 5 描边

描边是赛璐璐风格卡渲的另一大特点，凸显角色的同时，他同样也是使渲染效果更有“二次元味”的一块基石。

很喜欢崩三里那样带有自发光和透明的描边，加上Bloom后效果很好。顺带一提，12月7号是Kiana生日！庆生视频来不及做完了，只能在这里发一发QAQ

<img src="https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204211238584.png" alt="Kiana" style="zoom: 33%;" />

目前描边的方案大概有三类：第一类是基于屏幕后处理实现的，通过在后处理中根据法线和深度信息进行边缘检测来获取描边区域。其优点是逐像素的描边，精度更高，而缺点是在后处理中无法简单地灵活处理描边颜色这类信息。第二类是通过贴图来实现如膝盖区域的内描边，并且可以通过“本村线”这样的处理流程来减少锯齿，不过此方案只能处理内描边。第三类是在模型渲染过程中实现描边，即BackFacing通过额外的Pass来实现描边。优点是可以在对应Pass中灵活处理描边的表现，如区分区域控制描边颜色、透明度、自发光等，缺点是需要额外的Pass，并且是基于顶点实现的描边，并且需要特殊处理法线。

为了更灵活地控制描边效果，我使用的是BackFacing描边。基本原理是使用一个额外的Pass，在VertexShader中将模型沿着表面法线方向进行一定程度的外扩，在FragmentShader中对其颜色进行处理，并且整个Pass剔除模型的前半面，前半面显示的是原本的模型，后半面显示的是描边。简而言之，可以理解成渲染一个CullFront的额外放大后的模型。

![Outline Pass渲染的结果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204215310661.png)

但是，模型硬表面上的顶点，即具有多个法线的顶点，导入引擎后，引擎会将其视为具有各自不同法线朝向的不同顶点（模型导入引擎所见的顶点数往往比DCC软件中见到的顶点数多的原因），导致我们直接沿法线外扩后，硬边部位的描边会有断裂的情况。这个时候需要我们计算额外的平滑法线信息，用于外扩获得平滑的描边表现。我这一次处理平滑法线的方法参考（抄）了喵刀老师，借助Unity资产导入后处理来实现的，原理可以看文末链接的喵刀老师的文章，也可以去我的代码仓库的ModelSmoothNormalImporter.cs文件中查看我进一步注释后的喵刀老师的代码。

<img src="https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204214023274.png" alt="使用模型法线(左)和使用平滑法线(右)处理描边的效果" style="zoom:67%;" />

![使用模型法线(左)和使用平滑法线(右)处理描边的效果](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231204214313093.png)

平滑法线存储在法线空间中。因为人物模型会运动，所以在处理模型时，平滑法线信息需要转换到法线空间中进行存储（原理可以进一步查询大佬的资料），进而我们需要在shader中将平滑法线信息再还原回世界空间。这一步和处理法线贴图原理相同，流程更简单，只用在Vertex中处理法线，其代码如下：

```hlsl
input.vertexColor.r = input.vertexColor.r * 2.0 - 1.0;
input.vertexColor.g = input.vertexColor.g * 2.0 - 1.0;
float3 smoothNormalTS = normalize(float3(input.vertexColor.r, input.vertexColor.g,
    sqrt(1 - dot(float2(input.vertexColor.r, input.vertexColor.g), float2(input.vertexColor.r, input.vertexColor.g)))
        ));
```

因为法线外扩在VertexShader阶段实现，他的粗细是相对于模型本身而言的，如果不做处理，会在摄像机近时显得描边特别细，在摄像机远时显得描边特别粗，视觉效果不理想。所以描边粗细需要随着摄像机的距离进行调整，处理方式我参考（抄）了NiloToon的处理方式，代码如下：

```hlsl
float GetCameraFOV()
{
    //Shader中获取相机FOV
    //https://answers.unity.com/questions/770838/how-can-i-extract-the-fov-information-from-the-pro.html
    float t = unity_CameraProjection._m11;
    float Rad2Deg = 180 / 3.1415;
    float fov = atan(1.0f / t) * 2.0 * Rad2Deg;
    return fov;
}

float ApplyOutlineDistanceFadeOut(float distanceVS)
{
    //深度在ViewSpace中0-10范围内进行描边粗细的缩放
    #if UNITY_REVERSED_Z
        //DX平台
        return max(0.0, (50.0 - distanceVS) * 0.1);
    #else
        //OpenGL平台（还未做测试）
        return min(10.0, distanceVS) * 0.1;
    #endif
}
//距离控制描边粗细，改编自NiloToon的函数
float GetOutlineCameraFOVAndDistanceFixMultiplier(float positionVS_Z)
{
    float distanceFix;
    if(unity_OrthoParams.w == 0)
    {
        //透视相机
        float distance = abs(positionVS_Z);
        distanceFix = ApplyOutlineDistanceFadeOut(distance);
        distanceFix *= GetCameraFOV();
    }
    else
    {
        //正交相机
        float distance = abs(unity_OrthoParams.z);
        distanceFix = ApplyOutlineDistanceFadeOut(distance) * 50; //参考NiloToon的数值
    }

    return distanceFix;
}
```

为了实现不同区域的不同描边颜色，我们需要获取各个区域的遮罩。这个案例中我直接使用了ilmTexture贴图的A通道，即用于区分不同Ramp的灰度信息，在FragmentShader中，借Step处理并相减出不同区域的遮罩。但是处理的效果不够好，只实现了部分颜色可控，所以只把我的处理放在这里做一个参考（错误案例），代码如下：

```hlsl
half4 BackFaceOutlineFragment(Varyings input) : SV_Target
{
    //根据ilmTexture的五个分组区分出五个不同颜色的描边区域
    half outlineColorMask = SAMPLE_TEXTURE2D(_ilmTex, sampler_ilmTex, input.uv).a;
    half areaMask1 = step(0.003, outlineColorMask) - step(0.35, outlineColorMask);
    half areaMask2 = step(0.35, outlineColorMask) - step(0.55, outlineColorMask);
    half areaMask3 = step(0.55, outlineColorMask) - step(0.75, outlineColorMask);
    half areaMask4 = step(0.75, outlineColorMask) - step(0.95, outlineColorMask);
    half areaMask5 = step(0.95, outlineColorMask);
    half4 finalColor = areaMask1 * _OutlineColor1 + areaMask2 * _OutlineColor2 + areaMask3 * _OutlineColor3 + areaMask4 * _OutlineColor4 + areaMask5 * _OutlineColor5;

    return finalColor;
}
```

![描边Pass的表示，不带有描边的效果与带有描边的效果](E:/Document/Notes/Render/【Unity】GLR2.0-角色篇.assets/image-20231204222638335.png)



## 6 代码架构

![GLR2角色渲染相关代码架构](https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20231205233601528.png)



## 参考链接

YuiLu大佬的还原案例，是对我帮助最大的参考：

https://zhuanlan.zhihu.com/p/511150017

这篇文章里，作者更多地从本质上去思考如何把卡通渲染做得好看，探求卡通渲染的“法”这一层面的知识：

https://zhuanlan.zhihu.com/p/561494026

文如其题，作者在里面介绍了当今业界普遍使用的那些卡渲处理技术：

https://zhuanlan.zhihu.com/p/508826073

喵刀老师的基于Unity资产后处理和Job System实现的高效的平滑法线处理流程：

https://zhuanlan.zhihu.com/p/107664564

喵刀老师的屏幕空间深度边缘光教程：

 https://zhuanlan.zhihu.com/p/139290492

NiloToonURP管线案例，其中大佬对代码架构的清晰处理方式很值得学一学：

https://github.com/ColinLeung-NiloCat/UnityURPToonLitShaderExample



## 最后

再放一遍做的MMD，虽然效果还有欠缺，但是也花了很久时间去做了。如果文章对你有所帮助的话，也请转到视频点个赞和关注，救救我惨淡的数据吧！

https://www.bilibili.com/video/BV1ew41187H5/

后续还有后处理的使用，以及整个制作工作流的分享文章。

