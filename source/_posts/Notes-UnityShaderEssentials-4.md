---
title: 《Unity Shader入门精要》笔记（四）
date: 2023-04-01 00:00:03
tags:
 - Unity
 - Shader
 - TA
 - 《UnityShader入门精要》
categories: Notes
keywords: 'Unity Shader'
description: 噪声的使用、Unity中的渲染优化技术
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/100789898_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/41TN3CwhyZL._SY346_.jpg
katex: true
---

## NPR

因为我想将来重点研究NPR，所以NPR的部分统一放在之后的笔记整理



## 使用噪声

### 消融效果

#### 原理

概括来说就是**噪声纹理+透明度测试**，使用噪声纹理采样的结果和某个控制消融程度的阈值比较，如果小于阈值，就使用clip函数把它对应的像素裁剪掉，这些部分就对应“烧毁”的部分

#### Shader实现

- `Cull Off`命令关闭了shader的面片剔除，也就是说模型的正面和背面都会被渲染。因为消融会导致模型内部裸露，如果只渲染正面会出现错误的效果
- 为了能正确投射消融效果的阴影，使用了自定义的阴影投射Pass，在这里面通常会使用Unity内提供的内置宏V2F_SHADOW_CASTER\TRANSFER_SHADOW_CASTER_NORMALOFFSET和SHADOW_CASTER_FRAGMENT来帮助我们计算投射阴影时需要的各种变量，我们可以只关注自定义计算的部分

```glsl
Shader "ShaderLearning/Noise/Dissolve"
{
    Properties
    {
        _MainTex("Main Texture", 2D) = "white"{}
        _BumpMap("Bump Map", 2D) = "white"{}
        _DissolveTex("Dissolve Texture", 2D) = "black"{}
        _BurningLineWidth("Burning Line", Range(0,1)) = 0.5
        _BurnedAmount("Burned Amount", Range(0,1)) = 0.3
        _BurningFirstColor("Burning First Color", Color) = (1,1,1,1)
        _BurningSecondColor("Burning Second Color", Color) = (1,1,1,1)
        }
    Subshader
    {
        Pass
        {
            Tags {"LightMode"="ForwardBase"}
            
            Cull off
            
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #pragma multi_compile_fwdbase

            #include "Lighting.cginc"
            #include "AutoLight.cginc"
            #include "UnityCG.cginc"

            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;
            float4 _BumpMap_ST;
            sampler2D _DissolveTex;
            float4 _DissolveTex_ST;
            float _BurningLineWidth;
            float _BurnedAmount;
            fixed4 _BurningFirstColor;
            fixed4 _BurningSecondColor;

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 tangentLightDir : TEXCOORD0;
                float2 uv_base : TEXCOORD1;
                float2 uv_bump : TEXCOORD2;
                float2 uv_noise : TEXCOORD3;
                float3 worldPos : TEXCOORD4;//用来计算光照衰减
                SHADOW_COORDS(5)
            };

            v2f vert(appdata_tan v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv_base = TRANSFORM_TEX(v.texcoord,_MainTex);
                o.uv_bump = TRANSFORM_TEX(v.texcoord,_BumpMap);
                o.uv_noise = TRANSFORM_TEX(v.texcoord,_DissolveTex);

                TANGENT_SPACE_ROTATION;
                o.tangentLightDir = mul(rotation,ObjSpaceLightDir(v.vertex));
                o.worldPos = mul(unity_ObjectToWorld, v.vertex);
                TRANSFER_SHADOW(o);

                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                float burn = tex2D(_DissolveTex,i.uv_noise).r;

                clip(burn.r - _BurnedAmount);//剔除溶解掉的部分（值小则剔除）

                float3 tangentLightDir = normalize(i.tangentLightDir);
                float3 tangentNormal = UnpackNormal(tex2D(_BumpMap, i.uv_bump));//将法线信息从0 —— 1转换到 -1 —— 1

                //计算正常的漫反射
                fixed3 albedo = tex2D(_MainTex,i.uv_base);

                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;

                fixed3 diffuse = _LightColor0.rgb * albedo * saturate(dot(tangentLightDir,tangentNormal));

                //计算溶解效果（白色溶解：为了便于后续控制完全没溶解的情况）
                fixed t = 1 - smoothstep(0.0,_BurningLineWidth,burn-_BurnedAmount);
                t = pow(t,2);
                fixed3 burningColor = lerp(_BurningFirstColor,_BurningSecondColor,t);
                burningColor = pow(burningColor,5);//指数计算加强溶解对比效果

                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
                fixed3 finalColor = lerp(ambient + diffuse * atten, burningColor,t*step(0.0001,_BurnedAmount));//插值计算溶解部分和非溶解部分,step用于保证_BurnedAmount为0的时候没有溶解效果

                return fixed4(finalColor,1.0);
            }
            ENDCG
        }
        Pass    
        {
            Tags {"LightMode"="ShadowCaster"}
            CGPROGRAM
// Upgrade NOTE: excluded shader from DX11; has structs without semantics (struct v2f members uv_BurnMap)
#pragma exclude_renderers d3d11

            #pragma vertex vert
            #pragma fragment frag

            #pragma multi_compile_shadowcaster

            sampler2D _DissolveMap;
            float4 _DissolveMap_ST;
            float _BurnedAmount;

            struct v2f
            {
                V2F_SHADOW_CASTER;
                float2 uv_BurnMap;
            };

            v2f vert(appdata_base v)
            {
                v2f o;
                TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)

                o.uv_BurnMap = TRANSFORM_TEX(v.texcoord, _DissolveMap);

                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 burn = tex2D(_DissolveMap, i.uv_BurnMap).rgb;

                clip(burn.r - _BurnedAmount);

                SHADOW_CASTER_FRAGMENT(i)
            }
            
            ENDCG
            }
        }
    FallBack "Diffuse"
}
```



### 水波效果

#### 原理

原理与使用反射和折射模拟透明玻璃效果的原理基本一致。此处使用Cubemao作为环境纹理模拟反射效果；使用GrabPass获取当前屏幕的渲染，并使用法线对像素进行偏移模拟折射效果

#### Shader实现

- 其中将Queue设置成了Transparent，可以确保物体渲染时，其他所有不透明物体已经被渲染到屏幕上了
- 设置RenderType为Opaque是为了在使用着色器替换技术时，该物体可以在需要时被正确渲染（获取摄像机的深度和法线纹理时）
- 法线的扰动和计算是在世界空间下完成的（法线信息从世界空间转换到世界空间下进行计算），以便于对GrabPass读取到的贴图进行读取
- 使用了WaveXSpeed和WaveYSpeed两个属性，是为了模拟两层交叉的水面波动效果
- 计算偏移的屏幕坐标时，我们将偏移量和屏幕坐标的Z分量相乘，这是为了模拟深度越大，折射程度越大的效果

```glsl
Shader "ShaderLearning/Noise/WaterWave"
{
    Properties
    {
        _Color("Color Tint", Color) = (0,0.15,0.115,1.0)
        _MainTex("Base(RGB)", 2D) = "white"{}
        _WaveMap("Wave Map", 2D) = "white"{}
        _Cubemap("Environmental Cubemap", CUBE) = "_Skybox"{}
        _WaveSpeedX("Wave Speed X", Range(-0.1,0.1)) = 0.01
        _WaveSpeedY("Wave Speed Y", Range(-0.1,0.1)) = 0.01
        _Distortion("Distortion", Range(0,100)) = 10
    }
    
    Subshader
    {
        Tags{"Queue"="Transparent" "RenderType"="Opaque"}//Queue设置为半透明为了确保这个物体是在场景内物体渲染完成之后才被渲染的，表现出“透过水”的效果；而RenderType设置为Opaque是为了在使用着色器替换时物体可以在需要时被正确渲染
        
    	GrabPass{ "_RefractionTex" } //屏幕图像被存入RefractionTex中
        
        Pass
        {
            Tags { "LightMode"="ForwardBase" }
			
			CGPROGRAM
			
			#include "UnityCG.cginc"
			
			#pragma multi_compile_fwdbase
			
			#pragma vertex vert
			#pragma fragment frag
			
			fixed4 _Color;
			sampler2D _MainTex;
			float4 _MainTex_ST;
			sampler2D _WaveMap;
			float4 _WaveMap_ST;
			samplerCUBE _Cubemap;
			fixed _WaveSpeedX;
			fixed _WaveSpeedY;
			float _Distortion;	
			sampler2D _RefractionTex;
			float4 _RefractionTex_TexelSize;
			
			struct a2v {
				float4 vertex : POSITION;
				float3 normal : NORMAL;
				float4 tangent : TANGENT; 
				float4 texcoord : TEXCOORD0;
			};
			
			struct v2f {
				float4 pos : SV_POSITION;
				float4 scrPos : TEXCOORD0;
				float4 uv : TEXCOORD1;
				float4 TtoW0 : TEXCOORD2;  //由于是咋世界空间下计算法线
				float4 TtoW1 : TEXCOORD3;  
				float4 TtoW2 : TEXCOORD4; 
			};

			v2f vert (a2v v)
			{
				v2f o;
				o.pos = UnityObjectToClipPos(v.vertex);
				
				o.scrPos = ComputeGrabScreenPos(o.pos);//来获取对应被抓取屏幕图像的采样坐标
				
				o.uv.xy = TRANSFORM_TEX(v.texcoord, _MainTex);
				o.uv.zw = TRANSFORM_TEX(v.texcoord, _WaveMap);
				
				float3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;  
				fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);  
				fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);  
				fixed3 worldBinormal = cross(worldNormal, worldTangent) * v.tangent.w; 
				
				o.TtoW0 = float4(worldTangent.x, worldBinormal.x, worldNormal.x, worldPos.x);  
				o.TtoW1 = float4(worldTangent.y, worldBinormal.y, worldNormal.y, worldPos.y);  
				o.TtoW2 = float4(worldTangent.z, worldBinormal.z, worldNormal.z, worldPos.z);  
				
				return o;
            }

			fixed4 frag(v2f i) : SV_Target
			{
				float3 worldPos = float3(i.TtoW0.w,i.TtoW1.w,i.TtoW2.w);
				fixed3 viewDir = normalize(UnityWorldSpaceViewDir(worldPos));
				float2 speed = _Time.y * float2(_WaveSpeedX, _WaveSpeedY);

				//在切线空间下获取法线
				fixed3 bump1 = UnpackNormal(tex2D(_WaveMap, i.uv.zw + speed)).rgb;//
				fixed3 bump2 = UnpackNormal(tex2D(_WaveMap, i.uv.zw - speed)).rgb;//读取两次法线是为了能获取多层的水面波纹效果
				fixed3 bump = normalize(bump1 + bump2);

				//计算切线空间的偏移
				float2 offset = bump.xy * _Distortion * _RefractionTex_TexelSize.xy;
				i.scrPos.xy += offset * i.scrPos.z;//把偏移量和屏幕坐标相乘是为了模拟深度越大折射越强的效果
				fixed3 refrCol = tex2D(_RefractionTex, i.scrPos.xy/i.scrPos.w).rgb;//对scrPos进行透视除法后用于采样

				//从切线空间转换到世界空间
				bump = normalize(half3(dot(i.TtoW0.xyz,bump),dot(i.TtoW1.xyz,bump),dot(i.TtoW2.xyz,bump)));
				fixed4 texColor = tex2D(_MainTex, i.uv.xy + speed);
				fixed3 reflDir = reflect(-viewDir, bump);
				fixed3 reflCol = texCUBE(_Cubemap,reflDir).rgb * texColor.rgb * _Color.rgb;

				fixed fresnel = pow(1-saturate(dot(viewDir,bump)),4);
				fixed3 finalColor = reflCol * fresnel + refrCol * (1-fresnel);

				return fixed4(finalColor, 1);
				}
			ENDCG
        	}
    }
	Fallback Off
}
```



### 使用噪声纹理的灰度值生成法线信息

在纹理面板中把纹理类型设置为Normal Map，并选中Create from grayscale即可

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230613105817515.png"  />



## Unity中的渲染优化技术

### 移动平台的特点

移动设备上的GPU架构专注于尽可能使用更小的带宽和功能

- 为了减少overdraw的Tiled-based Deferred Rendering(TBDR)
- Early-Z技术剔除那些不需要被渲染的片元



### 影响性能的因素

**CPU主要保证帧率，GPU主要保证分辨率**

#### 性能瓶颈的原因

- CPU
  - 过多的draw call
  - 复杂的脚本或者物理模拟
- GPU
  - 顶点处理
    - 过多的顶点
    - 过多的逐顶点计算
  - 片元处理
    - 过多的片元（既可能是分辨率造成的，也可能是overdraw造成的）
    - 过多的逐片元计算
- 带宽
  - 使用了尺寸很大且未被压缩的纹理
  - 分辨率过高的帧缓存



#### 优化技术总览

- CPU优化
  - 使用批处理减少draw call数目
- GPU优化
  - 优化几何体
  - 使用LOD技术
  - 使用遮挡剔除（Occlusion Culling）技术
- 减少需要处理的片元数目
  - 控制绘制顺序
  - 警惕透明物体
  - 减少实时光照
- 减少计算复杂度
  - 使用shader的LOD技术
  - 代码方面的优化
- 节省内存带宽
  - 减少纹理大小
  - 利用分辨率缩放



### 减少draw call数目

处理思想：**每次调用draw call时尽可能多地处理多个物体**

什么物体可以一起处理：使用同一个材质的物体——对于使用同一个材质的物体，他们之间的区别仅仅在于顶点数据的差别，我们可以把这些顶点数据合并在一起，再一起发送给GPU，就可以完成一次批处理

- Unity支持两种批处理方式：动态批处理和静态批处理

- 不管是动态批处理还是静态批处理，它们的前提都是要使用**同一个材质**（是同一个，而不是使用了同一个shader的材质，相当于是UE中的同一个材质实例）

#### 动态批处理

**基本原理**：每一帧把可以进行批处理的mesh进行合并，再把合并后的模型数据传递给GPU，然后使用同一个材质进行渲染。

**优势**：实现方便，并且自由度高，经过批处理的物体仍然可以移动

**缺点**：只有满足条件的模型和材质才可以被动态批处理

主要的**限制条件**：

- 能够进行动态批处理的**网格的顶点属性规模要小于900**。比如说，如果shader中需要使用顶点位置、法线和纹理坐标这3个顶点属性，那么要想让模型能够被动态批处理，他的顶点数目不能超过300（数据具有即时性，只需要记住顶点属性规模这个概念，并且其会限制动态批处理即可）
- 使用lightmap的物体需要小心处理。这些物体需要额外的渲染参数，例如，在光照纹理上的索引、偏移量和缩放信息等。因此为了能让这些物体可以被动态批处理，我们需要保证他们指向光照纹理中的同一个位置
- 多Pass的shader会中断批处理。前向渲染中，我们有时需要使用额外的Pass来为模型添加额外的光照效果，这样一来模型就不会被动态批处理了。

#### 静态批处理

静态批处理适用于任何大小的几何模型

**实现原理**：只在运行开始阶段，把需要进行静态批处理的模型合并到一个新的网格结构中，这意味着这些模型不可以在运动时刻被移动。

**优势**：但是由于它只进行一次合并操作，因此比动态批处理高效

**缺点**：模型不可以在运动时刻被移动、需要占用更多的内存（存储合并的几何结构，其每一个物体会对应一个原本网格的复制体）

**操作**：只需要勾选物体的**Static**即可

#### 共享材质

如果两个材质之间只有使用的纹理不同，我们可以把这些纹理合并到一张更大的纹理中，即组成一张**图集（atlas）**。一旦使用了纹理，我们就可以使用同一个材质，再使用不同的采样坐标对纹理进行采样即可。



### 减少需要处理的顶点数目

#### 优化几何体

- 建模时就要记住，尽可能**减少模型中三角形面片的数目**，一些对于模型没有影响的、或是肉眼难以察觉到区别的顶点都要尽可能去掉

- unity中显示的顶点数目和三角形数目往往大于建模软件中所显示的数目，其原因是：
  - 建模软件更多地站在人的视角下理解顶点，而unity是在GPU的视角下理解顶点。GPU看来，有时需要把一个顶点拆分成多个顶点，原因主要有两个：一是为了**分离纹理坐标（uv splits）**，二是为了**产生平滑的边界（smoothing splits）**。两者的本质，其实都是因为对于GPU来说，顶点的每一个属性和顶点之间必须是一对一的关系。有些顶点的纹理坐标可能有多个（不同的面），这对GPU来说是不可理解的，因此它必须把这个顶点拆分成多个具有不同纹理坐标的顶点。平滑边也是类似，一个顶点可能会对应多个法线信息或切线信息，用于决定一个边是hard edge还是smooth edge
- 优化：
  - 减少顶点数目
  - 移除不必要的硬边以及纹理衔接，以此避免边界平滑和纹理分离

#### 模型的LOD

使用LOD Group组件为一个物体构建一个LOD



### 减少需要处理的片元数目

重点在于减少drawcall

#### 控制绘制顺序

由于深度测试的存在，如果我们能保证物体都是从**前往后绘制**的，那么就可以很大程度上减少overdraw

- unity中，**Queue数目小于2500**的(Background、Geometry和AlphaTest)的对象都会被认为是opaque的物体，这些物体总体上是从前往后绘制的。而其他的队列则是从后往前绘制。所以我们可以尽可能把物体的队列设置为不透明物体的渲染队列，而尽量避免使用半透明队列。
- 而且我们可以充分利用unity的渲染队列来控制绘制顺序：
  - 第一人称射击游戏中，对于游戏中的主要人物角色来说，他们使用的shader往往比较复杂，但是他们通常会**挡住屏幕的很大一部分区域**，因此可以**先绘制**他们（使用小的渲染队列）
  - 对于敌人，他们通常会出现在各种掩体之后（**任务渲染比场景复杂，且部分身体部位被遮挡**，但是没有完全遮住，所以不会被剔除），因此可以在所有常规的不透明物体后面渲染他们
  - 对于天空盒，它几乎覆盖了所有像素，而且它**一定是在所有物体的后面**，因此它的队列可以设置为“**Geometry+1**”，这样可以保证不会因为天空盒造成overdraw

#### 时刻警惕透明物体

- 对于半透明物体来说，由于他没有开启深度写入，如果想要获取正确的效果，就必须从后往前渲染，这意味着半透明物体几乎一定会造成overdraw

- 移动平台上，AlphaTest也会影响性能。虽然他没有关闭深度写入，但是由于他的实现使用了discard或clip操作，这些操作会导致一些硬件的优化策略失效。
  - 例如，TBDR技术会在执行fragment shader之前就判断哪些瓦片被真正地渲染。但是由于透明度测试在片元着色器中使用了discard函数，改变了片元是否会被渲染的结果，只有在执行了所有的fragment shader之后，GPU才会知道哪些片元会被真正渲染到屏幕上。因此GPU就无法使用这个优化策略了。这种时候，使用透明度混合的性能往往比使用透明度测试更好。

#### 减少实时光照和阴影

- 对于逐像素光源来说，被这些光源照亮的物体需要再被渲染一次，并且他们会中断批处理
  - 例如，一个场景里如果包含了3个逐像素的点光源，而且使用了逐像素的shader，那么很可能将drawcall的次数提升3倍（CPU的瓶颈），同时也会增加overdraw（GPU的瓶颈）
- 实时阴影也是一个非常消耗性能的效果，不仅是CPU需要提交更多的draw call，GPU也要进行更多处理
- 优化
  - 使用烘焙lightmap作为替代，这样运行时只需要更具纹理采样即可
  - 移动平台上，一个物体使用的逐像素光源数目应小于1（不包括平行光），如果要使用更多的平行光，可以选择用逐顶点光照来代替



### 节省带宽

#### 减少纹理大小

- 所有纹理的**长宽比最好相同**（正方形），而且**长宽值最好是2的整数幂**，因为有很多优化策略只有在这种时候才可以发挥最大效用。

- 尽可能**使用mipmap**（原理见百人计划2.9-深入理解GPU硬件架构及运行机制）
- 压缩纹理也可以节省带宽，只需要把压缩纹理格式设置为自动压缩即可

#### 利用分辨率缩放

性能和画面之间的trade off



### 减少计算复杂度

#### Shader中的LOD技术

与之前提到的模型的LOD技术类似，Shader的LOD技术可以控制使用的shader等级，其原理是，只有shader的LOD值小于某个设定的值，这个shader才会被使用，而使用了超过设定值的shader的物体不会被渲染

通常在SubShader中使用类似下面的语句指明该shader的LOD值：

```glsl
SubShader {
    Tags { "RenderType" = "Opaque"}
    LOD 200
}
```

#### 代码方面的优化

- 尽可能使用低精度的浮点值进行计算
  - float/highp适用于存储诸如顶点坐标等变量，但他的计算速度使最慢的，我们应该尽量避免在fragment  shader中使用这种精度进行计算
  - half/mediump适用于一些标量、纹理坐标等变量，它的计算速度约是float的两倍
  - fixed/lowp适用于绝大多数颜色变量和归一化后的方向矢量，他的计算速度越是float的4倍
    - 但要尽量避免对这些低精度变量进行频繁的swizzle操作(如color.xwxw)
    - 应当尽量避免在不同精度之间的转换，这有可能造成一定程度的性能下降
- 对于绝大多数GPU来说，使用插值寄存器把数据从vertex shader传给下一个阶段时，应该使用尽可能少的插值变量（把多个变量组合成一个）（PowerVR除外）
- 尽可能不要使用全屏幕的后处理效果
- 尽可能不要使用分支语句和循环语句
- 尽可能避免使用类似sin、tan、pow、log等较为复杂的数学运算，可以使用查找表进行替代
- 尽可能不要使用discard操作，这会影响硬件的某些优化（TBDR）



## NEXT

Book：

《Real-time Rendering》

《Physically based rendering：From theory to implementation》

《GPU Gem》1-3

《GPU Pro》1-7

Course：

Physically Based Shading in Theory and Practice

Advances in Real-Time Rendering
