---
title: 《Unity Shader入门精要》笔记（一）
date: 2023-04-01 00:00:00
tags:
 - Unity
 - Shader
 - TA
 - 《UnityShader入门精要》
categories: Notes
keywords: 'Unity Shader'
description: 此章包含一些shader基础和透明效果
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/100789898_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/41TN3CwhyZL._SY346_.jpg
katex: true
---

## 血与泪水

> 一些自己在学习过程中花了很久才去解决的bug，记录一下加深印象

- Properties不要乱改属性的名字，有些特定的属性名会和Fallback的其他shader有些关系，避免用不规范的属性名带来一些未知bug
- 如果shader没有报错，先去看看vertex和fragment有没有被有效编译
  - 比如说#pragma vertex vert有没有被有被写成#pragma vretex vert
- a2v结构体中获取的UV坐标TEXCOORD0用float2接收即可，且在TRANSFORM_TEX时就有多个需要控制纹理缩放，都是共用那同一个TEXCOORD的UV坐标

**一些注意事项：**

1. ComputeScreenPos和ComputeGrabScreenPos获得的srcPos在用于读取贴图前需要进行透视除法

   - ComputeScreenPos：使用摄像机的深度和法线纹理时
   - ComputeGrabScreenPos：使用GrabPass时

   ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230610162319552.png)

2. 顶点动画如果在模型空间下进行计算需要禁用批处理，但因此会带来一定的性能下降，增加Draw Call，因此我们应该尽量避免在模型空间下用一些绝对位置和方向进行计算

3. 顶点动画需要我们自己提供shadowcaster以实现正确的阴影投射效果



## 渲染流水线

### 应用阶段(CPU)

**输出渲染所需的几何信息(渲染图元rendering primitives)**

1. 将数据从硬盘加载到RAM，再加载到VRAM上

2. 设置渲染状态

3. 调用Draw Call

   - 由CPU发起，GPU接收

   - 命令仅仅指向一个需要被渲染的图元列表

   - 通过命令缓冲区实现CPU与GPU的并行（改变渲染状态的命令比Draw Call的调用更加耗时）

   - Draw Call不宜过多

     - 原因：每次调用Draw Call之前，CPU需要向GPU发送很多数据以改变渲染状态，并且还需要完成很多工作（例如检查渲染状态）

     - 解决：Batch Rendering（使用同一种渲染状态，渲染多个模型）

       ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Untitled.png)



### 几何阶段(GPU)

**输出屏幕空间的二维顶点坐标、每个顶点对应的深度值、着色等相关信息**

1. Vertex Shader
   - **每个顶点**都会调用一次Vertex Shader
   - 顶点之间的Vertex Shader**相互独立**，即无法得到顶点之间的关系
   - 工作主要是坐标变化和逐顶点光照
     - 坐标变换：模型空间到齐次裁剪空间
2. 曲面细分着色器
   - 可选
   - 细分图元
3. 几何着色器
   - 作用就是将一个图元变换为另一个完全不同的图元，或者修改图元的位置
4. Clipping
   - 用于处理部分在视野内的图元
   - 在视野外部的顶点应该使用一个新的顶点来代替，顶点位于交界处
5. 屏幕映射
   - 输出**窗口坐标系**（屏幕坐标+z坐标）



### 光栅化阶段(GPU)

**决定每个渲染图元中的哪些像素应该被绘制在屏幕上**

1. 三角形设置
   - 计算三角网格表示数据的过程（计算每条边上的像素坐标，用于得到整个三角形网格对像素覆盖情况）
2. 三角形遍历（扫描变换）
   - 检查像素是否被三角形覆盖，若覆盖则生成一个fragment
   - 一个fragment != 一个像素
     - fragment是包含了很多**状态的集合**，包括但不限于屏幕坐标、深度信息、法线、纹理坐标等
3. Fragment Shader
   - 输出一个或多个**颜色值**
   - 每个Fragment之间**相互独立**，Fragment Shader仅可以影响单个Fragment
4. 逐片元操作
   - 解释
     - Per-Fragment Operation（OpenGL）+Output-Merge（DirectX）
   - 主要任务
     1. 决定片元的可见性（很多测试工作，如Depth Test、Stencil Test）
     2. 如果片元通过了所有的测试，则与Color Buffer中的颜色进行混合



## Vert/Frag Shader基础结构

### 一段简易的Vertex + Fragment Shader

```glsl
Shader "Custom/UnityShader/Chapter5-SimpleShader"//当前shader的路径及名称
{
    Properties//属性--->材质面板的可调节参数
    {
        _Color ("Color Tint",Color) = (1.0, 1.0, 1.0, 1.0)//Properties语义块支持的属性
    }
    SubShader 
    {
        Pass {
            CGPROGRAM
						
			//编译指令
            #pragma vertex vert//告诉Unity哪个函数包含了vertex/fragment shader的代码
            #pragma fragment frag

            fixed4 _Color;//需要定义一个与属性名称和变量类型都匹配的变量 变量类型匹配表

            //自定义的结构体，用于存放传入vertex shader的变量信息
			//a2v = Appliacation to Vertex Shader
            struct a2v
            {
                float4 vertex : POSITION;// Unity支持的常用语义
                float3 normal : NORMAL;
                float4 texcoord : TEXCOORD0;
            };

            //v2f = Vertex Shader to Fragment Shader
            struct v2f
            {
                float4 pos:SV_POSITION;
                float3 color:COLOR0;
            };

			//vertex shader代码
            v2f vert(a2v v)
            {
                v2f o;//声明输出结构
                o.pos = UnityObjectToClipPos(v.vertex);//UnityCG.cginc中的帮助函数
                o.color = v.normal*0.5 + fixed3(0.5,0.5,0.5);
                return o;
            }
			//fragment shader代码
            float4 frag(v2f i):SV_Target
            {
                fixed3 c = i.color;
                c *= _Color.rgb;
                return fixed4(c,1.0);
            }
            ENDCG
        }
    }
}
```

### 附录1：Properties语义块支持的属性类型

|    属性类型     |                使用举例                |
| :-------------: | :------------------------------------: |
|       Int       |          _Int("Int", Int) = 2          |
|      Float      |      _Float("Float", Float) = 1.5      |
| Range(min, max) | _Range("Range", Range(0.0, 0.5)) = 3.0 |
|      Color      |   _Color("Color", Color) = (1,1,1,1)   |
|     Vector      | _Vector("Vector", Vector) = (2,3,4,5)  |
|       2D        |          _2D("2D", 2D) = ""{}          |
|      cube       |    _Cube("Cube", Cube) = "white"{}     |
|       3D        |       _3D("3D", 3d) = "black"{}        |

### 附录2：ShaderLab属性类型与CG变量类型匹配表

|   ShaderLab   |          CG           |
| :-----------: | :-------------------: |
| Color, Vector | float4, half4, fixed4 |
| Range, Float  |  float, half, fixed   |
|      2D       |       sampler2D       |
|     Cube      |      samplerCube      |
|      3D       |       sampler3D       |

### 附录三： Unity支持的常用语义

#### 应用阶段传递模型数据到顶点着色器：

|   语义    |                             描述                             |
| :-------: | :----------------------------------------------------------: |
| POSITION  |              模型空间中的顶点位置，通常为float4              |
|  NORMAL   |                    顶点法线，通常为float3                    |
|  TANGENT  |                    顶点法线，通常为float4                    |
| TEXCOORDn | 顶点的纹理坐标，TEXCOORD表示第一组纹理坐标，通常为float2或float4 |
|   COLOR   |                顶点颜色，通常为fixed4或float4                |

#### 顶点着色器到片元着色器：

|        语义         |                 描述                 |
| :-----------------: | :----------------------------------: |
|     SV_POSITION     |        裁剪空间中的顶点坐标，        |
|       COLOR0        | 通常用于输出第一组顶点的颜色，非必须 |
|       COLOR1        | 通常用于输出第二组顶点的颜色，非必须 |
| TEXCOORD0~TEXCOORD7 |     通常用于输出纹理坐标，非必须     |

#### 片元着色器输出

|   语义    |              描述              |
| :-------: | :----------------------------: |
| SV_Target | 输出值将会存储到render targe中 |



## 基础光照模型：Blinn-Phong

### 原理解析

Color = Ambient + Diffuse + Specular

- Ambient

  由Lighting.cginc中的函数`UNITY_LIGHTMODEL_AMBIENT`直接获取

- Diffuse

  逐像素的Lambert/half Lambert，在fragment中进行计算

  Diffuse颜色=光照颜色\*漫反射颜色\*[0-1]范围的光线与表面法线的点积

- Specular

  逐像素的高光计算，在fragment中进行计算

  Specular颜色=光照颜色\*高光反射颜色\*max(0,光线、视线的中间向量与表面法线的点积)

  > 因为对高光不用进行右侧钳制，所以用max钳制左侧即可，且节省性能



### 代码演示

```glsl
Shader "ShaderLearning/BasicLighting/BlingPhongShadingModel"
{
    Properties
    {
        _Diffuse("Diffuse Color",Color) = (1,1,1,1)
        _Specular("Specular Color",Color) = (1,1,1,1)
        _Glossy("Glossy",Range(8.0,256.0)) = 20.0
    }

    SubShader
    {
        pass
        {
            Tags {"LightMode"="ForwardBase"}

            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            #include "Lighting.cginc"

            fixed4 _Diffuse;
            fixed4 _Specular;
            float _Glossy;
            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };
            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(_Object2World,v.vertex).xyz;
                return o;
            }

            fixed4 frag(v2f i) : SV_TARGET
            {
				//计算环境光
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz;
				//计算世界空间下的光照方向
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
				//计算漫反射颜色
                fixed3 diffuse = _Diffuse.rgb * _LightColor0.rgb * saturate(dot(worldLightDir,i.worldNormal));
				//计算高光反射颜色
                fixed3 viewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                fixed3 halfDir = normalize(worldLightDir + viewDir);
                fixed3 specular = _Specular.rgb * _LightColor0.rgb * pow(max(0,dot(halfDir,i.worldNormal)),_Glossy);
				//返回求和颜色
                return fixed4((ambient + diffuse + specular),1.0);
            }
            ENDCG
        }
    }
    FallBack "Specular"
}
```

### UnityCG.cginc中常用的帮助函数

官方文档：[Unity - Manual: Built-in shader helper functions (unity3d.com)](https://docs.unity3d.com/Manual/SL-BuiltinFunctions.html)

| 函数名                                         | 输入                 | 返回                               | 是否normalize |
| ---------------------------------------------- | -------------------- | ---------------------------------- | ------------- |
| float3 UnityWorldSpaceViewDir(float3 v)        | 世界空间下的顶点位置 | 世界空间中从该点到摄像机的观察方向 | 否            |
| float3 ObjSpaceViewDir(float4 v)               | 模型空间下的顶点位置 | 模型空间中从该点到摄像机的观察方向 | 否            |
| float3 UnityWorldSpaceLightDir(float3 v)       | 世界空间下的顶点位置 | 世界空间中从该点到光源的光照方向   | 否            |
| float3 ObjSpaceLightDir(float4 v)              | 模型空间下的法线方向 | 模型空间中从该点到摄像机的光照方向 | 否            |
| float3 UnityObjectToWorldNormal(float3 normal) | 模型空间下的法线方向 | 世界空间中的法线方向               | 是            |
| float3 UnityObjectToWorldDir(float3 dir)       | 模型空间下的方向矢量 | 世界空间中的方向矢量               | 是            |
| float3 UnityWorldToObjectDir(float3 dir)       | 世界空间中的方向矢量 | 模型空间下的方向矢量               | 是            |



## 纹理基础

### 纹理的基础用法

```glsl
Shader "ShaderLearning/Texture/MaskTex"
{
    Properties
    {
        _Color("Color Tint",Color) = (1,1,1,1)
        _Specular("Specular Color",Color) = (1,1,1,1)
        _Gloss("Specular Gloss",Range(8.0,256)) = 20.0
        _MainTex("Main Texture",2D) = "white"{}//贴图的shaderlab属性为2D
        _BumpTex("Normal Texture",2D) = "bump"{}
        _BumpScale("Bump Scale",float) = 1.0//用于控制法线的强度
        _MaskTex("Mask Texture",2D) = "white"{}
        _MaskScale("Mask Scale",float) = 1.0//用于控制遮罩的强度
    }
    SubShader
    {
        pass
        {
            Tags {"LightMode"="ForwardBase"}
            CGPROGRAM
            #pragma vertex vert 
            #pragma fragment frag 
            #include "Lighting.cginc"

            float _Gloss;
            float4 _Color;
            float4 _Specular;
            sampler2D _MainTex;
            sampler2D _BumpTex;
            sampler2D _MaskTex;
            float4 _MainTex_ST;//需要使用纹理名_ST的方式来声明某个纹理的属性，ST=Scale+Translation,其中x,y表示Scale，z,w表示Transition
            float _BumpScale;
            float _MaskScale;

						//此段为在切线空间中计算光照模型

            struct a2v
            {
                float4 vertex : POSITION;
                float2 texcoord : TEXCOORD;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;//是四维向量，w通道存储了副切线bitangent的方向信息
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
                float3 lightDir : TEXCOORD1;
                float3 viewDir : TEXCOORD2;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = TRANSFORM_TEX(v.texcoord,_MainTex);//进行贴图的缩放，宏内部调用了之前声明的纹理的_ST属性

                TANGENT_SPACE_ROTATION;//为UnityCG.cginc中的宏定义，等价于以下两行：
								//fixed3 Binormal = cross(normalize(v.normal),normalize(v.tangent.xyz)) * v.tangent.w;
								//float3x3 rotation = float3x3(v.tangent.xyz, binormal, v.normal);
								//用于得到从物体空间转换到切线空间的变换矩阵TBN矩阵
                o.lightDir = mul(rotation,ObjSpaceLightDir(v.vertex)).xyz;//左乘TBN矩阵：world->obj
                o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;

                return o;
            }

            fixed4 frag(v2f i) : SV_TARGET
            {
                fixed3 tangentLightDir = normalize(i.lightDir);
                fixed3 tangentViewDir = normalize(i.viewDir);
                fixed3 tangentNormal = UnpackNormal(tex2D(_BumpTex,i.uv));//读取法线贴图信息并将映射法线信息
                tangentNormal.xy *= _BumpScale;//设置法线强度
                tangentNormal.z = sqrt(1-dot(tangentNormal.xy, tangentNormal.xy));//计算法线z向量的信息

                fixed3 albedo = tex2D(_MainTex,i.uv).rgb * _Color;

                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;

                fixed3 diffuse = _LightColor0.rgb * albedo * saturate(dot(tangentNormal,tangentLightDir));

                fixed specularMask = tex2D(_MaskTex,i.uv)*_MaskScale;//读取遮罩贴图并设置遮罩强度
                fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);
                fixed3 specular = _LightColor0.rgb * _Specular * pow(max(0,dot(halfDir,tangentNormal)),_Gloss) * specularMask;

                return fixed4(ambient + diffuse + specular,1.0);
            }
            ENDCG
        }
    }
    FallBack "Specular"
}
```



### 法线贴图

#### 凹凸映射

两种实现凹凸映射的方法：

- 表面位移displacement：通过高度纹理来模拟，得到一个修改后的法线值
  - 计算复杂，不能直接得到表面法线，而是需要由像素灰度值计算
- 法线纹理normal map：直接存储表面法线

#### 法线纹理的解释

- 存储空间：
  - 每个顶点都有一个属于自己的切线空间，法线纹理往往存储在模型顶点的**切线空间**中
- 为什么大部分是蓝色：
  - 每个法线方向所在的坐标空间是不一样的，即是在表面各自的切线空间中。这种法线纹理实际上就是存储了每个点在各自的切线空间中的法线扰动方向。法线贴图中存储的法线，在经过映射后大多指向屏幕之外，所以法线贴图大多是蓝紫色调的。

#### 法线纹理的反映射（解压）

法线纹理中存储的就是表面的法线方向，但是由于法线的分量范围是[-1,1]，而像素的分量范围是[0,1]，因此需要一个映射将法线信息存储到法线纹理中。并且在shader中对法线纹理采样后，还需要对结果进行**反映射**来得到原本的法线信息。

将纹理的纹理类型标识为Normal map，对采样出的法线信息调用`UnpackNormal`函数即可完成映射

#### 法线纹理属性 —— Create from Grayscale

- Bumpness：控制纹理凹凸程度
- Filtering：
  - Smooth：生成后的法线较为平滑
  - Sharp：使用Sobel滤波（用于边缘检测的滤波器）来生成法线
    - Sobel滤波的实现：在一个3x3滤波器中计算x和y方向的导数，然后从中获取法线（考虑水平方向和竖直方向上的像素差）

#### 在不同空间中计算光照模型

- 在切线空间下计算光照模型（如上[示例代码](###纹理的基础用法)]）

- 在世界空间下计算光照模型：

```diff
struct a2v
            {
                float4 vertex : POSITION;
-               float2 texcoord : TEXCOORD;
+				float4 texcoord : TEXCOORD0;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
-               float2 uv : TEXCOORD0;
+				float4 uv : TEXCOORD0;
-               float3 lightDir : TEXCOORD1;
-               float3 viewDir : TEXCOORD2;
+				float4 T2W0 : TEXCOORD1;
+				float4 T2W1 : TEXCOORD2;
+				float4 T2W2 : TEXCOORD3;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
-               o.uv = TRANSFORM_TEX(v.texcoord,_MainTex);//进行贴图的缩放，宏内部调用了之前声明的纹理的_ST属性
+				o.uv.xy = TRANSFORM_TEX(v.texcoord, _MainTex);
+				o,uv.zw = TRANSFORM_TEX(v.vertex, _BumpMap);

-               TANGENT_SPACE_ROTATION;						
-               o.lightDir = mul(rotation,ObjSpaceLightDir(v.vertex)).xyz;//左乘TBN矩阵：world->obj
-               o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex)).xyz;

+				float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
+				float3 worldNormal = UnityObjectToWorldNormal(v.normal);
+				fixed4 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);
+				fixed3 worldBitangent = cross(worldNormal, worldTanget)*v.tangent.w;//tangent切线中的w通道存储了副切线bitangent的方向信息

+				o.T2W0 = float4(worldTangent.x, worldBitangent.x, worldNormal.x, worldPos.x);
+				o.T2W1 = float4(worldTangent.y, worldBitangent.y, worldNormal.y, worldPos.y);
+				o.T2W2 = float4(worldTangent.z, worldBitangent.z, worldNormal.z, worldPos.z);
                return o;
            }

            fixed4 frag(v2f i) : SV_TARGET
            {
-               fixed3 tangentLightDir = normalize(i.lightDir);
-               fixed3 tangentViewDir = normalize(i.viewDir);
-               fixed3 tangentNormal = UnpackNormal(tex2D(_BumpTex,i.uv));
-               tangentNormal.xy *= _BumpScale;//设置法线强度
-               tangentNormal.z = sqrt(1-dot(tangentNormal.xy, tangentNormal.xy));

-                fixed3 albedo = tex2D(_MainTex,i.uv).rgb * _Color;
-                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
-                fixed3 diffuse = _LightColor0.rgb * albedo * saturate(dot(tangentNormal,tangentLightDir));
-               fixed specularMask = tex2D(_MaskTex,i.uv)*_MaskScale;
-               fixed3 halfDir = normalize(tangentLightDir + tangentViewDir);
-               fixed3 specular = _LightColor0.rgb * _Specular * pow(max(0,dot(halfDir,tangentNormal)),_Gloss) * specularMask;

+				float3 worldPos = float3(i.T2W0.w, i.T2W1.w, i.T2W2.w);
+               fixed3 lightDir = normalize(UnityWorldSpaceLightDir(worldPos));
+               fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));
                //获取法线信息
+               fixed3 bump = UnpackNormal(tex2D(_BumpMap,i.uv.zw));
+               bump.xy *= _BumpScale;//因为法线是单位矢量，所以可以通过xy来计算出z，同样也就可以通过缩放xy来控制z的强度（法线贴图表示的是法线的扰动）
+               bump.z = sqrt(1-saturate(dot(bump.xy , bump.xy)));
+               bump = normalize(half3(dot(i.T2W0.xyz,bump),dot(i.T2W1.xyz,bump),dot(i.T2W2.xyz,bump)));
                //读取贴图
+               fixed3 albedo = tex2D(_MainTex,i.uv.xy) * _Color;
+               fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;
+               fixed3 diffuse = _LightColor0.rgb * albedo * saturate(dot(lightDir,bump));
+               fixed3 halfDir = normalize(viewDir + lightDir);
+               fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0,dot(halfDir,bump)),_Gloss);

                return fixed4(ambient + diffuse + specular,1.0);
            }
```

> 两者的对比： 在切线空间中计算光照模型，需要在vertex shader中将视线向量和光线向量转换到切线空间 在世界空间中计算光照模型，需要在vertex shader中将normal和binormal转换到世界空间，以此获得法线空间到世界空间的变换矩阵(BTN矩阵)，并通过插值寄存器传递矩阵，再在fragment shader中将法线信息从切线空间转换到世界空间



### Ramp贴图

使用贴图控制漫反射光照，得到NPR的效果：

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/Untitled2.png" alt="Untitled2" style="zoom: 67%;" />

Ramp贴图：

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials3.png)

```glsl
fixed4 frag(v2f i) : SV_TARGET
{
		fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
		fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));

        fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * _Color;
        fixed3 halfDir = normalize(worldLightDir + worldViewDir);
        fixed3 specular = _LightColor0.rgb * _Color.rgb * pow(max(0,dot(halfDir,i.worldNormal)),_Gloss);
                
        fixed halfLambert = 0.5 * saturate(dot(i.worldNormal,worldLightDir)) + 0.5;
        fixed3 diffuseColor = tex2D(_RampTex,fixed2(halfLambert,halfLambert)) * _Color.rgb;//Key Code->根据光照计算结果读取Ramp图
        fixed3 diffuse = diffuseColor * _LightColor0.rgb;

        return fixed4(ambient + diffuse + specular, 1.0);
}
```



## 透明效果

两种实现方法：

- Alpha Test透明度测试
  - 阈值控制，简单直接，不透明或完全透明
  - 面片草效果
- Alpha Blend透明度混合
  - 可以得到真正的透明效果

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials4.png" alt="UnityShaderEssentials4" style="zoom: 50%;" />



### Unity Shader的渲染顺序 —— Queue

Unity提前定义的5个渲染队列：

|    名称     | 队列索引号 |                             描述                             |
| :---------: | :--------: | :----------------------------------------------------------: |
| Background  |    1000    |               在其他任何队列之前渲染，用于背景               |
|  Geometry   |    2000    |                  默认的渲染队列，不透明物体                  |
|  AlphaTest  |    2450    | 需要使用透明度测试的物体（Unity5中分出来的，因为这用渲染更加高效） |
| Transparent |    3000    |   按从后往前的顺序，使用了透明度混合的物体都应该使用该队列   |
|   Overlay   |    4000    |              实现一些叠加效果，在最后渲染的物体              |

写法实例：`Tags {”Queue”=”Transparent”}`



可通过在值后加数字的形式来重新指定渲染队列：

自定义渲染序列：`Tags {”Queue”=”Transparent + 1”}`

> 染队列会通过重复绘制直接影响性能，合理的队列可以极大提升渲染效率 Unity中，队列小于2500的对象都会被认为是不透明物体，从前往后绘制；而其他的队列则是从后往前绘制。这意味着我们需要尽可能将队列设置为不透明物体的渲染队列，尽量避免重复绘制



### AlphaTest

#### Single Side(Cull back side)

```glsl
Shader "ShaderLearning/Transparency/AlphaTest"
{
    Properties
    {
        _Color("Color Tint",Color) = (1,1,1,1)
        _MainTex("Main Texture",2D) = "white"{}//通过贴图的A通道来控制透明度
        _CutThreshold("Cut Alpha Threshold Value",Range(0,1)) = 0.5//通过一个阈值来控制裁剪的alpha值
    }

    SubShader
    {
        Tags {"Queue"="AlphaTest" "IgnoreProjector"="True" "RenderType"="TransparentCutout"}
        pass
        {
            Tags{"LightMode"="ForwardBase"}

            //...

            fixed4 frag(v2f i) : SV_TARGET
            {
                //...

                fixed4 textureColor = tex2D(_MainTex,i.uv);

                //Alpha Test
                clip(textureColor.w-_CutThreshold);//比阈值小的就直接不渲染，比阈值大的作为不透明物体渲染
                //Equale to:
                //if(textureColor.w-_CutThreshold < 0.0){discard;}

                //...

                return fixed4(ambient + diffuse, 1.0);
            }
            ENDCG
        }
    }
    FallBack "Transparent/Cutout/VertexLit"
}
```



#### Both Side

在pass内禁用Cull即可：

```glsl
Pass
{
		//关闭cull
		Cull Off
}
```



### AlphaBlend

#### Without ZWrite

关闭ZBuffer写入后，因为无法进行深度排序，所以渲染的半透明效果可能是错误的

```glsl
Shader "ShaderLearning/Transparency/AlphaBlend"
{
    Properties
    {
        _Color("Color Tint",Color) = (1,1,1,1)
        _MainTex("Main Texture",2D) = "white"{}//通过贴图的A通道来控制透明度
        _AlphaScale("Alpha Scale",Range(0,1)) = 1//控制Alpha的强弱
    }
    SubShader
    {
        Tags{"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent" }
        pass
        {
            Tags {"LightMode"="ForwardBase"}

            ZWrite Off
            Blend SrcAlpha OneMinusSrcAlpha

           //...

            fixed4 frag(v2f i) : SV_TARGET
            {
                //...

                return fixed4(ambient + diffuse, texColor.a * _AlphaScale);//通过贴图和AlphaScale来控制返回值的a通道
            }
            ENDCG
        }
    }
    FallBack "Transparent/VertexLit"
}
```

#### With ZWrite

这样得到的结果不会表现模型内部之间的半透明效果，只有整个物体的半透明效果

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials5.png)

==在写入颜色缓冲前，多一个Pass来写入深度信息而不进行实际的渲染==

```glsl
Shader "ShaderLearning/Transparency/AlphaBlendWithZWrite"
{
   //...
    SubShader
    {
        Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
        pass
        {
            ZWrite on
            ColorMask 0//ColorMask设置颜色通道的写掩码，可以写入RGBA的任意组合，当ColorMask设置为0时意味着该Pass不写入任何颜色通道，即不会输出任何颜色，用来单独做Z测试写入ZBuffer
        }

        pass
        {
            Tags {"LightMode"="ForwardBase"}
            ZWrite Off
            Blend SrcAlpha OneminusSrcAlpha

            //...同AlphaTest without ZWrite
        }
    }
    FallBack "Transparency/VertexLit"
}
```

#### Both Side

实现半透明效果，但是动用了两个Pass先后渲染前后面

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials6.png)

==核心在于分两个Pass，且先渲染背面再渲染前面==

```glsl
Shader "ShaderLearning/Transparency/AlphaBlendBothSides"
{
    Properties
    {
        _Color("Color Tint",Color) = (1,1,1,1)
        _MainTex("Main Texture",2D) = "white"{}
        _AlphaScale("Alpha Scale",Range(0,1)) = 1
    }

    SubShader
    {
        Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}

        BlendOp add
        Blend SrcAlpha OneMinusSrcAlpha
        ZWrite Off

        pass
        {
            Tags {"LightMode"="ForwardBase"}
            Cull front 

            CGPROGRAM
            //...
            ENDCG
        }

        pass
        {
            Tags {"LightMode"="ForwardBase"}
            Cull back

            CGPROGRAM
						//...
            ENDCG
        }
    }
    Fallback "Transparenct/VertexLit"
}
```



### ShaderLab的混合命令

#### 混合操作BlendOperation

注：源颜色Src指的是frag中计算出来的颜色，目标颜色Dst指的是Color Buffer中读出的颜色

| BlendOperation |                             功能                             |
| :------------: | :----------------------------------------------------------: |
|      Add       |                     默认的混合指令，相加                     |
|      Sub       |   相减，$O_{rgb}=SrcFractor*S_{rgb}+DstFractor * D_{rgb} $   |
|     RevSub     | 反向相减，$O_{rgb}=DstFractor * O_{rgb} - SrcFractor * S_{rgb}$ |
|      Min       |           取目标颜色和源颜色的最小值，混合因子无效           |
|      Max       |           去目标颜色和源颜色的最大值，混合因子无效           |

#### 设置混合因子

|                       设置                       | 解释                                       |
| :----------------------------------------------: | ------------------------------------------ |
|            Blend SrcFactor DstFactor             | RGBA通道使用同样的混合因子                 |
| Blend SrcFactor DsrFactor, SrcFactorA DstFactorA | RGB通道使用前两个混合因子，A通道使用后两个 |

#### 混合因子

|     混合因子     |                             解释                             |
| :--------------: | :----------------------------------------------------------: |
|       One        |                           因子为1                            |
|       Zero       |                           因子为0                            |
| SrcColor/DtColor | 因子为源颜色/目标颜色，如果用于混合RGB，则用SrcColor的RGB去混合；如果混合A，则用SrcColor的A去混和 |
| SrcApha/DstColor |                因子为源颜色/目标颜色的Alpha值                |
|    OneMinus—     |                             1-x                              |

