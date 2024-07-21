---
title: 《Unity Shader入门精要》笔记（二）
date: 2023-04-01 00:00:01
tags:
 - Unity
 - Shader
 - TA
 - 《UnityShader入门精要》
categories: Notes
keywords: 'Unity Shader'
description: 光照进阶、纹理进阶
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/100789898_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/41TN3CwhyZL._SY346_.jpg
katex: true
---



## 光照进阶

### Unity的渲染路径

#### LightMode标签支持的渲染路径设置选项

`Tags { "LightMode" = "----"}`

|              标签              |          渲染路径          |                         描述                         |
| :----------------------------: | :------------------------: | :--------------------------------------------------: |
|             Always             |            所有            |        该Pass总是被渲染，但是不会计算任何光照        |
|          ForwardBase           |          前向渲染          | 环境光、最重要的平行光、逐顶点光源/SH光源、Lightmaps |
|           ForwardAdd           |          前向渲染          |                   额外的逐像素光照                   |
|            Deferred            |          延迟渲染          |                    渲染到GBuffer                     |
|          ShadowCaster          |         前向、延迟         |   将物体的深度信息渲染到shadowmap或一张深度纹理中    |
|          PrepassBase           |  (Legyacy)遗留的延迟渲染   |             渲染法线eh高光反射的指数部分             |
|          PrepassFinal          |   (Legacy)遗留的延迟渲染   |    通过合并纹理、光照和自发光来渲染得到最后的颜色    |
| Vertex、VertexLMRGBM、VertexLM | (Legacy)遗留的顶点照明渲染 |                                                      |

Unity中的前向渲染

- 前向渲染中有3中处理光照（照亮物体）的方式：逐顶点处理、逐像素处理、SH（Spherical Harmonics）球谐函数处理

  如何决定一个光源使用哪种处理方式：

  - 光源类型
  - 光源的渲染模式：Important/Not Important

- 前向渲染中的2种Pass

  ![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials7.png)

  - Base Pass
    - 渲染的光源默认支持阴影
    - 渲染最重要的平行光
    - 自发光和环境光也在其中，因为只希望计算一次
  - Additional Pass
    - 渲染的光源默认没有阴影效果，但是可以使用`#pragma multi_compile_fwdadd_fullshadows`代替`#pragma multi_compile_fwdadd`来获得阴影效果
    - 开启和设置了混合模式，与上一次的光照结果在帧缓存中进行叠加，通常选择Blend One One
  - 对于前向渲染，一个Unity Shader通常会定义一个Base Pass和一个Additional Pass，一个Base Pass只会被执行一次；而一个Additional Pass会根据影响该物体的**其他逐像素光源**的数目被多次调用，即每个逐像素光源会执行一个Additional Pass（通常是4个逐像素光源）

#### Unity中的顶点照明渲染

通常在一个Pass中就可以完成对物体的渲染，最多访问到8个逐顶点光源

- 要求最少、运算性能最高
- 效果最差
- 仅仅是前向渲染的一个子集

#### Unity中的延迟渲染

- 原理：
  - 第一个Pass中不进行任何的光照计算，仅仅计算哪些片元可见（深度缓冲技术），当发现一个片元可见，就把他的相关信息存储到GBuffer中；
    - 漫反射颜色、高光反射颜色、平滑度、法线、自发光和深度信息
    - 对每个物体来说，这个Pass**只执行一次**
  - 然后，第二个Pass中利用GBuffer中的各个片元信息进行**真正的光照计算**
- **每个光源**都可以按**逐像素**的方式处理
- 默认的GBuffer包含以下几个Render Texture：
  - TR
  - RT0：RGB存漫反射颜色，A没有被使用
  - RT1：RGB存高光反射颜色，A存高光反射指数
  - RT2：RGB存法线，A没有被使用
  - RT3：存自发光+Lightmap+反射探针Reflection Probe
  - 深度缓冲和模板缓冲



#### 前向渲染代码示例

```glsl
Shader "ShaderLearning/AdvancedLighting/ForwardRendering"
{
    Properties
    {
        _ColorTint("Color Tint",Color) = (1,1,1,1)
        _SpecularColor("Specular Color",Color) = (1,1,1,1)
        _Gloss("Gloss",Range(8.0,256.0)) = 20.0
    }
    SubShader
    {
        pass
        {
            //Base Pass: Directional light & Ambient light
            Tags {"LightMode"="ForwardBase"}
            CGPROGRAM
            #pragma multi_compile_fwdbase //声明变体，用于保证unity可以为对应类型的pass生成所有需要的shader变种

            #pragma vertex vert 
            #pragma fragment frag 
            #include "Lighting.cginc"

            fixed4 _ColorTint;
            fixed4 _SpecularColor;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldPos : TEXCOORD0;
                float3 worldNormal : TEXCOORD1;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldPos = mul(unity_ObjectToWorld,v.vertex.xyz);
                o.worldNormal = UnityObjectToWorldNormal(v.normal.xyz);
                return o;
            }
            fixed4 frag(v2f i) : SV_TARGET
            {
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos));
                fixed3 worldNormal = normalize(i.worldNormal);

                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * _ColorTint.rgb;
                fixed3 diffuse = _LightColor0.rgb * _ColorTint.rgb * saturate(dot(worldLightDir,worldNormal));

                fixed3 halfDir = normalize(worldLightDir + normalize(_WorldSpaceCameraPos.xyz - i.worldPos));
                fixed3 specular = _LightColor0.rgb * _SpecularColor.rgb * pow(max(0,dot(worldNormal,halfDir)),_Gloss);

                fixed atten = 1.0;//定向光光照衰减始终为1
                return fixed4(ambient + (diffuse + specular)*atten,1.0);
            }
            ENDCG
        }

        pass
        {
            //Additional Pass: Other Pixel Light
            Tags{"LightMode"="ForwardAdd"}
            Blend One One//混合模式为one one

            CGPROGRAM
            #pragma multi_compile_fwdadd //声明变体为forwardadd
            #pragma vertex vert 
            #pragma fragment frag 
            #include "Lighting.cginc"
            #include "AutoLight.cginc"

            fixed3 _ColorTint;
            fixed3 _SpecularColor;
            float _Gloss;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldPos : TEXCOORD0;
                float3 worldNormal : TEXCOORD1;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex.xyz);
                o.worldNormal = UnityObjectToWorldNormal(v.normal.xyz);
                return o;
            }
            fixed4 frag(v2f i) : SV_TARGET
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                #ifdef USING_DIRECTIONAL_LIGHT
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz);
                #else
                    fixed3 worldLightDir = normalize(_WorldSpaceLightPos0.xyz - i.worldPos);
                #endif

                fixed3 diffuse = _LightColor0.rgb * _ColorTint.rgb * saturate(dot(worldLightDir,worldNormal));
                fixed3 halfDir = normalize(worldLightDir + normalize(UnityWorldSpaceViewDir(i.worldPos)));
                fixed3 specular = _LightColor0.rgb * _SpecularColor.rgb * pow(max(0,dot(halfDir,worldNormal)),_Gloss);

                //计算光照衰减
                #ifdef USING_DIRECTIONAL_LIGHT
					fixed atten = 1.0;//如果是定向光则光照衰减为1
				#else
                    //使用一张纹理作为查找表（Look Up Table--LUT）来获得光照衰减值
					#if defined (POINT)//对点光源
                        //首先需要得到该点在光源空间的位置来读取LightTexture中的光照衰减值
				    float3 lightCoord = mul(unity_WorldToLight, float4(i.worldPos, 1)).xyz;
                        //使用光源空间中顶点距离的平方来对纹理采样
                            //光源空间以光源为原点
                            //只查询LightTexture的对角线上的值（通过光源视角的深度去查询2D纹理）
				    fixed atten = tex2D(_LightTexture0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;
				    #elif defined (SPOT)//对聚光灯
				    float4 lightCoord = mul(unity_WorldToLight, float4(i.worldPos, 1));
				    fixed atten = (lightCoord.z > 0) * tex2D(_LightTexture0, lightCoord.xy / lightCoord.w + 0.5).w * tex2D(_LightTextureB0, dot(lightCoord, lightCoord).rr).UNITY_ATTEN_CHANNEL;
				    #else
				        fixed atten = 1.0;
				    #endif
				#endif

                return fixed4((diffuse + specular)*atten,1.0);
            }
            ENDCG
        }
    }
    Fallback "Specular"
}
```



### 衰减与阴影

#### 光照衰减

- 用于光照衰减的纹理（如上代码实例内的Look-up Table）

- 使用数学公式计算衰减值

  ```glsl
  fixed distance = lengh(_WorldSpaceLightPos0.xyz - i.worldPos.xyz);
  atten = 1.0 / distance; //线性衰减
  ```

#### 阴影

- 阴影的实现

  - Shadow Map：将摄像机放到光源的位置，摄像机看不到的地方就是阴影

  - SSM（ScreenSpace Shadow Map）

    根据光源的阴影映射纹理和摄像机的深度纹理来获得屏幕空间的阴影图（将摄像机深度转换到光源空间，与光源的阴影映射纹理进行比较，如果转换过去的深度更高则说明这个表面可见但是处于光源的阴影中，以此来得到屏幕空间的阴影图）

- 一个物体接收其他物体投射的阴影，以及向其他物体投射阴影是两个过程

  - 接收阴影：在shader中对阴影映射纹理（包括屏幕空的阴影图）进行采样，把采样结果和最后的光照结果相乘来产生阴影效果
  - 投射阴影：把该物体加入光源的阴影映射纹理的计算中，从而让其他物体在对阴影映射纹理采用时可以得到该物体的相关信息。在Unity中，这个过程是通过为该物体执行LightMode为ShadowCaster的Pass来实现的。如果使用SSM，Unity还会使用这个Pass生成一张摄像机的深度纹理。

#### 阴影的实现

- **让物体投射阴影：**

  - builtin-shaders-xxx->DefaultResourcesExtra->Normal-VertexLit.shader中的Shadow Caster：

    实际目的就是把深度信息写入RenderTarget中

  ```glsl
  // Pass to render object as a shadow caster
  Pass {
      Name "Shadow Caster"
      Tags { "LightMode" = "ShadowCaster"}
      
      CGPROGRAM
      #pragma vertex vert
      #pragma fragment frag
      #pragma multi_compile_shadowCaster
      #include "UnityCG.cginc"
          
      struct v2f {
          V2F_SHADOW_CASTER;
      }
      
      v2f vert( appdata_base v) 
      {
          v2f o;
          TRANSFER_SHADOW_CASTER_NORMALOFFSET(o)
          return o;
      }
      
      float4 frag( v2f i ) : SV_Target
      {
          SHADOW_CASTER_FRAGMENT(i)
      }
      ENDCG
  }
  ```

- **让物体接收阴影：**使用阴影三剑客**SHADOW_COORDS**、**TRANSFER_SHADOW**、**SHADOW_ATTENUATION**

  - 首先需要在vertex shader的输出结构体v2f中添加一个内置的宏**SHADOW_COORDS**：

    ```glsl
    struct v2f {
        float4 pos : SV_POSITION;
        float3 worldNormal : TEXCOORD0;
        float3 worldPos : TEXCOORD1;
        SHADOW_COORDS(2)
    }
    ```

    宏内部所填数字表示存贮阴影贴图纹理坐标所用的插值寄存器，一般使用下一个未被使用的插值寄存器

  - 然后在vertex shader返回前添加内置宏**TRANSFER_SHADOW**：

    ```glsl
    v2f vert ( a2v v )
    {
        v2f 0;
        ...
        // Pass shadow coordinates to pixel shader
            TRANSFER_SHADOW(o);
        
        return 0;
    }
    ```

    这个宏用于在顶点着色器中计算上一步中声明的阴影纹理坐标

  - 接着在fragment shader中计算阴影值使用内置宏**SHADOW_ATTENUATION**：

    ```glsl
    //Use shadow coordinates to sample shadow map
    fixed shadow = SHADOW_ATTENUATION(i);
    ```

  - 注意：为了确保宏能正确工作，需要保证自定义的变量名和这些宏中使用的变量名相同：a2v结构体中的顶点坐标变量名必须是**vertex**，顶点着色器的输入结构体v2f必须命名为**v**，且v2f中的顶点位置必须命名为**pos**

#### 使用内置函数实现衰减和阴影

```glsl
//宏UNITY_LIGHT_ATTENUATION既计算了光照衰减atten又计算了阴影shadow,并且融合在最终的atten中
UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);
```

宏UNITY_LIGHT_ATTENUATION源码：

![自己定义的destName同时包含了衰减和阴影](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials8.png)

#### AlphaBlendShadow

```glsl
Shader "ShaderLearning/AdvancedLighting/AlphaBlendShadow"
{
    Properties
    {
        _Color("Color Tint", Color) = (1,1,1,1)
        _Specular("Specular Color", Color) = (1,1,1,1)
        _Gloss("Gloss", Range(8,256)) = 20
        _AlphaScale("Alpha Scale", Range(0,1)) = 1
        _MainTex("Main Texture",2D) = "white"{}
    }
    SubShader
    {
        Tags {"Queue"="Transparent" "IgnoreProjector"="True" "RenderType"="Transparent"}
        BlendOp add 
        Blend SrcAlpha OneMinusSrcAlpha
        ZWrite Off
        
        UsePass "Legacy Shaders/VertexLit/SHADOWCASTER"
        
        pass
        {
            Tags {"LightMode"="ForwardBase"}
            Cull front 

            CGPROGRAM
            #pragma multi_compile_fwdbase
            #pragma vertex vert 
            #pragma fragment frag 
            #include  "Lighting.cginc"
            #include "AutoLight.cginc"

            fixed4 _Color;
            fixed4 _Specular;
            float _Gloss;
            float _AlphaScale;
            sampler2D _MainTex;
            float4 _MainTex_ST;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float2 texcoord : TEXCOORD0;
            };
            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
                float2 uv : TEXCOORD2;
                SHADOW_COORDS(3)
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
                TRANSFER_SHADOW(o);
                return o;
            }

            fixed4 frag(v2f i) : SV_TARGET
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos)).xyz;
                fixed4 texColor = tex2D(_MainTex, i.uv);
                fixed3 albedo = texColor.rgb * _Color.rgb;
                
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;

                fixed3 diffuse = _LightColor0.rgb * albedo * saturate(dot(worldNormal,worldLightDir));

                fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                fixed3 halfDir = normalize(worldViewDir + worldLightDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0,dot(worldNormal,halfDir)),_Gloss);

                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);


                fixed3 color = ambient + (diffuse + specular)*atten;

                return fixed4(color,texColor.a*_AlphaScale);
            }

            ENDCG
        }

        pass
        {
            Tags {"LightMode"="ForwardBase"}
            Cull back

            CGPROGRAM
            #pragma multi_compile_fwdbase nolightmap nodirlightmap nodynlightmap novertexlight
            #pragma multi_compile_RECEIVE_SHADOW_ON
            #pragma vertex vert 
            #pragma fragment frag 
            #include  "Lighting.cginc"
            #include "AutoLight.cginc"

            fixed4 _Color;
            fixed4 _Specular;
            float _Gloss;
            float _AlphaScale;
            sampler2D _MainTex;
            float4 _MainTex_ST;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float2 texcoord : TEXCOORD0;
            };
            struct v2f
            {
                float4 pos : SV_POSITION;
                float3 worldNormal : TEXCOORD0;
                float3 worldPos : TEXCOORD1;
                float2 uv : TEXCOORD2;
                SHADOW_COORDS(3)
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.worldNormal = UnityObjectToWorldNormal(v.normal);
                o.worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                o.uv = TRANSFORM_TEX(v.texcoord, _MainTex);
                TRANSFER_SHADOW(o);
                return o;
            }

            fixed4 frag(v2f i) : SV_TARGET
            {
                fixed3 worldNormal = normalize(i.worldNormal);
                fixed3 worldLightDir = normalize(UnityWorldSpaceLightDir(i.worldPos)).xyz;
                fixed4 texColor = tex2D(_MainTex, i.uv);
                fixed3 albedo = texColor.rgb * _Color.rgb;
                
                fixed3 ambient = UNITY_LIGHTMODEL_AMBIENT.xyz * albedo;

                fixed3 diffuse = _LightColor0.rgb * albedo * saturate(dot(worldNormal,worldLightDir));

                fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(i.worldPos));
                fixed3 halfDir = normalize(worldViewDir + worldLightDir);
                fixed3 specular = _LightColor0.rgb * _Specular.rgb * pow(max(0,dot(worldNormal,halfDir)),_Gloss);

                UNITY_LIGHT_ATTENUATION(atten, i, i.worldPos);

                fixed shadow = SHADOW_ATTENUATION(i);

                fixed3 color = ambient + (diffuse + specular)*atten*shadow;

                return fixed4(color,texColor.a*_AlphaScale);
            }
            ENDCG
        }
    }
    Fallback "Transparent/VertexLit"
}
```





## 纹理进阶

### Cubemap与环境反射、折射

#### 在Unity中创建Cubemap

```C##
//使用Unity提供的Camera.RenderToCubemap完成
void OnWizardCreate () {
	//创建用于渲染的摄像机
    GameObject go = new GameObject( "CubemapCamera");
	go.AddComponent<Camera>();
    //将摄像机放置到待渲染位置renderFromPosition对象的位置
	go.transform.position = renderFromPosition.position;
    //渲染Cubemap
	go.GetComponent<Camera>().RenderToCubemap(cubemap);
		
	//销毁临时创建的摄像机
	DestroyImmediate( go );
}
```

1. 创建一个空的GameObject对象作为渲染到位置
2. 新建一个专门用于存储的立方体纹理（$\text{Create}\rightarrow\text{Legacy}\rightarrow\text{Cubemap}$），同时为了让脚本能够顺利将图像渲染到该立方体纹理中，需要在他的面板中勾选**Readable**选项
3. 从Unity菜单栏选择$\text{GameObejct}\rightarrow\text{Render into Cubemap}$，打开脚本中实现的用于渲染立方体纹理的窗口，并把低1步的GameObject和第2步的创建的Cubemap分别拖拽到窗口中的**Render From Position**和**Cubemap**
4. 单击**Render!**按钮即可

![](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230610135337391.png)

<img src="http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230610135446285.png" alt="image-20230610135446285" style="zoom:50%;" />

需要注意的是，我们需要为Cubemap设置大小，即**Face Size**选项，Face Size值越大，渲染出来的立方体纹理分辨率越大，可能效果越好，但是需要占用的内存越大



#### 环境映射

##### 反射

```glsl
//1.需要在Properties内声明Cube类型的属性
Properties
{
		_Cubemap("Reflection Cubemap" ,3D) = "_Skybox"{} //3D类型的属性的默认值为"_Skybox"{}
}

//2.在CG代码中声明该属性
CGPROGRAM
samplerCUBE _Cubemap;

//3.为了节省性能，我们在vertex内计算反射的光线，通过插值计算器将反射的方向传递给fragment。所以需要在v2内声明反射方向
struct v2f
{
		float3 worldRefl : TEXCOORDn;
};

//4.在vertex中利用reflect函数计算世界空间下的反射方向
v2f vert(a2v v)
{
		//...
		o.worldRefl = reflect(-o.worldViewDir, o.worldNormal);
		//reflect函数第一个参数为入射光的方向，第二个参数为法线所在直线
		//入射方向和反射方向是同向的，即如果入射方向是指向入射点，那么反射方向就是由入射点指出。所以此处worldViewDir要加负号
}

//5.在fragment中读取CUBE贴图
fixed4 frag(v2f i) : SV_Target
{
		//...
		fixed3 reflection = texCUBE(_Cubemap, i.worldRefl).rgb;
}
```

##### 折射

```glsl
//与反射大同小异，唯一的区别在于vertex中函数调用的不同
v2f vert(a2v v)
{
		//...
		o.worldRefr = refract(-normalize(o.worldViewDir), normalize(o.worldNormal), _RefractRatio);
		//refrect函数第一个参数为入射光的方向，第二个参数为法线所在直线，第三个参数为折射率
		//入射方向和折射方向是同向的，即如果入射方向是指向入射点，那么折射方向就是由入射点指出。所以此处worldViewDir要加负号
}
```

##### 菲涅尔反射

**万物均有菲涅尔**

![菲涅尔的常见计算模型](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials9.png)

```glsl
//本质上就是在frag内通过菲涅尔系数对环境反射内容与漫反射内容之间进行插值计算
fixed4 frag(v2f i) : SV_Target
{
		//...
		fixed fresnel = _FresnelScale + (1 - _FresnelScale) * pow(1 - dot(worldNormal, worldViewDir), 5);
		fixed3 color = ambient + lerp(diffuse, reflection, saturate(fresnel)) * atten;
}
```



### Render Texture渲染纹理

#### RT的概念

现代GPU允许我们把整个三维场景渲染到一个中间缓冲中，即**渲染目标纹理（Render Target Texture，RTT）**，而不传统的帧缓存或后备缓冲区(back buffer)

与之相关的是**多重目标渲染（Multiple Render Target，MRT）**，这种技术指的是GPU允许我们把场景同时渲染到多个渲染目标纹理中，而不再需要为每个渲染目标纹理单独渲染完整的场景。延迟渲染就是使用多重目标渲染的一个应用

#### 创建RT

Unity为RTT定义了一种专门的纹理类型——**渲染纹理RenderTexture**，Unity中渲染纹理通常有以下两种使用方式：

- 在Project目录下创建一个渲染纹理，将某个摄像机的渲染目标设置为该渲染纹理，这样摄像机的渲染结果就会实时更新到RT中，而不会显示到屏幕
- 在屏幕后处理中使用**GrabPass**命令或**OnRenderImage**函数来获取当前屏幕图像，unity会将这个屏幕图像放到一张和屏幕分辨率相同的渲染纹理中，然后可以在自定义的Pass中将他们当作普通的纹理来处理，进而实现各种屏幕特效



#### 使用RenderTexture制作镜子

在处理代码前需要了先创建一个渲染纹理，并且把它指定到用于捕获的摄像机上

![UnityShaderEssentials10](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/UnityShaderEssentials10.png)

接下来编写镜子的shader

```glsl
		
Shader "ShaderLearning/AdvancedTexture/Mirror"
{
		//在Properties内声明纹理属性，它对应了由镜子摄像机渲染得到的渲染纹理
    Properties
    {
        _MainTex("Main Texture", 2D) = "white"{}
    }

    Subshader
    {
        pass
        {
            CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag

            struct a2v
            {
                float4 vertex : POSITION;
                float2 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float2 uv : TEXCOORD0;
            };

            sampler2D _MainTex;

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                o.uv = v.texcoord;
				//镜子对称需要翻转u轴
                o.uv.x = 1 - o.uv.x;
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                return tex2D(_MainTex, i.uv);
            }
            ENDCG
        }
    }
}
```



### GrabPass

#### GrabPass的概念

- 除去RenderTexture外，在Unity中，我们还可以使用一种特殊的Pass来完成获取屏幕图像的目的——GrabPass

- 当我们在Shader中定义了一个GrabPass后，Unity会把当前屏幕渲染的图像绘制在一张纹理中，以便我们在后续的Pass中访问它。
- 通常会使用GrabPass来实现诸如玻璃等透明材质的模拟，相比于普通的透明混合，GrabPass可以对物体后面的图像进行更加复杂的处理，例如用法线来模拟折射
- 需要注意的是：在使用GrabPass时需要额外**小心物体的渲染队列设置**。GrabPass通常用于渲染透明物体，所以我们需要把渲染队列设置为Transparent（即`Queue = “Transparent”`），这样才能保证渲染该物体时所有的不透明物体已经被渲染到屏幕上了，以获取正确的屏幕图像



#### 使用GrabPass制作玻璃效果

使用法线模拟折射效果

```glsl
//1.设置渲染队列和GrabPass
//必须要设置为Transparent的渲染队列，否则深度排序会出错
Tags {"RenderType"="Opaque" "Queue"="Transparent"}
        
GrabPass { "_RefractionTex" }//定义一个抓取屏幕的Pass，在之后的Pass中就可以使用_RefractionTex

//2.通过法线模拟折射
//在vertex中需要获取当前物体在摄像机中的位置，用于读取当前物体范围内的图像
struct v2f
{
		//...
		float4 scrPos : TEXCOORD4;
};

v2f vert(a2v i)
{
		//...
		o.scrPos = ComputeScreenPos(o.pos);
}

//3.在frag中制作扭曲效果
//获取带有扭曲效果的折射颜色
fixed4 frag (v2f i) : SV_Target
{
    ...
    fixed2 offset = bump.xy * _Distortion * _RefractionTex_TexelSize.xy * 100;//通过法线来扭曲折射贴图（乘100加强视觉效果）
	//_Refraction_TexelSize可以获取该纹理的纹素大小
	i.scrPos.xy = offset * i.scrPos.z + i.scrPos.xy;//将扭曲应用到屏幕纹理坐标上，此处多乘的srcPos.z可以让形变程度随着距离摄像机的远近发生变化
	fixed3 refrCol = tex2D(_RefractionTex,i.scrPos.xy/i.scrPos.w).rgb;//使用扭曲后的屏幕纹理坐标坐标读取贴图，通过ComputeScreenPos得到的是线性空间下的坐标，在最后用于读取贴图前需要做透视除法
    ...
}

```

![ComputeScreenPos获得的srcPos在用于读取贴图前需要进行透视除法](http://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/image-20230610162319552.png)

完整代码：

```glsl
Shader "ShaderLearning/AdvancedTexture/GlassRefraction"
{
    Properties
    {
        _MainTex("Main Texture", 2D) = "white"{}
        _BumpMap("Normal Texture", 2D) = "white"{}
        _Cubemap("Environment Cubemap", Cube) = "_skybox"{}
        _Distortion("Distortion", Range(0,100)) = 10
        _RefractAmount("Refract Amount", Range(0,1)) = 1
    }
    Subshader
    {
        //必须要设置为Transparent的渲染队列，否则深度排序会出错
        Tags {"RenderType"="Opaque" "Queue"="Transparent"}
        
        GrabPass { "_RefractionTex" }//定义一个抓取屏幕的Pass，在之后的Pass中就可以使用_RefractionTex
        
        Pass
        {
            CGPROGRAM

            #pragma vertex vert
            #pragma fragment frag

            #include "UnityCG.cginc"

            sampler2D _MainTex;
            float4 _MainTex_ST;
            sampler2D _BumpMap;
            float4 _BumpMap_ST;
            samplerCUBE _Cubemap;
            float _Distortion;
            fixed _RefractAmount;
            sampler2D _RefractionTex;
            float4 _RefractionTex_TexelSize;

            struct a2v
            {
                float4 vertex : POSITION;
                float3 normal : NORMAL;
                float4 tangent : TANGENT;
                float2 texcoord : TEXCOORD0;
            };

            struct v2f
            {
                float4 pos : SV_POSITION;
                float4 uv : TEXCOORD0;
                float4 T2W0 : TEXCOORD1;
                float4 T2W1 : TEXCOORD2;
                float4 T2W2 : TEXCOORD3;
                float4 scrPos : TEXCOORD4;
            };

            v2f vert(a2v v)
            {
                v2f o;
                o.pos = UnityObjectToClipPos(v.vertex);
                
                o.uv.xy = TRANSFORM_TEX(v.texcoord, _MainTex);
                o.uv.zw = TRANSFORM_TEX(v.texcoord, _BumpMap);

                //Get the TangentSpace2WorldSpace Matrix
                fixed3 worldNormal = UnityObjectToWorldNormal(v.normal);
                fixed3 worldTangent = UnityObjectToWorldDir(v.tangent.xyz);
                fixed3 worldBinormal = cross(worldNormal,worldTangent)*v.tangent.w;
                fixed3 worldPos = mul(unity_ObjectToWorld, v.vertex).xyz;
                o.T2W0 = fixed4(worldTangent.x, worldBinormal.x, worldTangent.x, worldPos.x);
                o.T2W1 = fixed4(worldTangent.y, worldBinormal.y, worldTangent.y, worldPos.x);
                o.T2W2 = fixed4(worldTangent.z, worldBinormal.z, worldTangent.z, worldPos.x);

                //得到对应被抓取的屏幕图像的纹理坐标
                o.scrPos = ComputeScreenPos(o.pos);
                
                return o;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                fixed3 worldPos = fixed3(i.T2W0.w, i.T2W1.w, i.T2W2.w);
                fixed3 worldViewDir = normalize(UnityWorldSpaceViewDir(worldPos));

                //Get Bump texture in tangent space
                fixed3 bump = UnpackNormal(tex2D(_BumpMap, i.uv.zw));
                //获取带有扭曲效果的折射颜色
                fixed2 offset = bump.xy * _Distortion * _RefractionTex_TexelSize.xy * 100;//通过法线来扭曲折射贴图,乘100加强一下视觉效果
                i.scrPos.xy = offset * i.scrPos.z + i.scrPos.xy;//将扭曲应用到屏幕纹理坐标上，
                fixed3 refrCol = tex2D(_RefractionTex,i.scrPos.xy/i.scrPos.w).rgb;//使用扭曲后的纹理坐标读取贴图
                //获取反射颜色
                bump = normalize(half3(dot(i.T2W0.xyz,bump),dot(i.T2W1.xyz,bump),dot(i.T2W2.xyz,bump)));
                fixed3 reflDir = reflect(-worldViewDir,bump);
                fixed4 texColor = tex2D(_MainTex,i.uv.xy);
                fixed3 reflCol = texCUBE(_Cubemap,reflDir).rgb * texColor.rgb;
                //通过插值计算颜色
                fixed3 finalColor = reflCol * (1 - _RefractAmount) + refrCol * _RefractAmount;

                return fixed4(finalColor, 1.0);
            }
            ENDCG
            }
    }
    FallBack "Diffuse"
}
```



### RenderTexture vs. GrabPass

- GrabPass的好处在于实现简单，只需要在shader中加几行代码即可
- 从效率上讲，使用渲染纹理的效率往往要好过GrabPass，尤其是在移动设备上。
  - 渲染纹理可以自定义渲染纹理的大小，尽管这种方法需要把部分场景再次渲染一遍，但我们可以通过调整摄像机的渲染层来减少二次渲染时的场景大小，或使用其他办法来控制摄像机是否需要开启。而使用GrabPass获得到的图像分辨率和显示屏幕是一致的，这意味着在一些高分辨率的设备上可能会造成严重的**带宽影响**。而且在移动设备上，GrabPass虽然不会重新渲染场景，但它往往需要CPU直接读取后背缓冲（Back Buffer）中的数据，**破坏了CPU和GPU之间的并行性**



### 其他抓屏方法

Unity5中引入了**命令缓冲（Command Buffers）**来允许我们拓展Unity的渲染流水线。

使用Command Buffer我们也可以获得类似抓屏的效果，它可以在不透明物体渲染后把当前的图像复制到一个临时的渲染目标纹理中，然后在那里进行一些额外的操作（例如模糊等），最后把图像传递给需要使用它的物体进行处理和显式。除此之外，Command Buffer还允许我们实现很多特殊的效果。

官方文档：[Unity - Manual: Extending the Built-in Render Pipeline with CommandBuffers (unity3d.com)](https://docs.unity3d.com/Manual/GraphicsCommandBuffers.html)
