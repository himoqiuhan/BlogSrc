---
title: 技术美术百人计划学习笔记（先行版-基础渲染光照介绍）
date: 2023-06-14 11:18:25
tags: 
 - Shader
 - TA
 - 百人计划
categories: Notes
keywords: '技术美术百人计划'
description: 
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/97358036_p0_master1200.jpg
cover: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/TA100.png
swiper_index: 1 
katex: true
---

## [先行版]基础渲染光照介绍

## Lambert

漫反射：lambert数值为法线和光照的点乘

```glsl
float3 worldLight = normalize(_WorldSpaceLightPos0.xyz)
float3 worldNormal = normalize(i.worldNormal)
//...
float NdotL = max(0.0, dot(finiNormal, worldLight))
```

## Phong

环境光+漫反射+高光

高光：光照颜色*高光反射颜色*视线方向与反射方向的点乘的gloss次幂

Gloss用于控制高光反射范围

```glsl
float3 reflectDir = normalize(reflect(-worldLight, finiNormal));//计算反射方向
float3 viewDir = normalzie(_WorldSpaceCameraPos.xyz - i.worldPOs.xyz);//计算视角方向（该像素的位置）
float VdotR = max(0.0, dor(reflectDir, viewDir);
float3 specular = _LightColor0.rgb * _Specular.rgb * pow(VdotR, _Gloss);
//...
float3 color = diffuse + ambient + specular;
```

## Blinn-Phong

与Phong模型的区别在于计算specular时，使用的是半角向量而非反射向量

比起Phong模型，计算效率提升至少三分之一以上

```glsl
float3 halfDir = normalize(worldLight + viewDir);
float3 NdotH = saturate(dot(halfDir, viewDir);
float specular = _LightColor0.rgb * _Specular.rgb * pow(NdotH, _Gloss);
```

## 法线贴图

UnpackNormal：空间转换（【0-1】空间→【-1-1】空间）

Unity内置的法线转换：

```glsl
inline fixed3 UnpackNormalDXT5nm(fixed4 packednormal)
{
	fixed3 normal;
	normal.xy = packednormal.xy * 2 - 1;
	normal.z = sqrt(1 - saturate(dot(normal.xy, normal.xy)));
	return normal;
}

inline fixd3 UnpackNormal(fixed4 packednormal)
{
##ifdef UNITY_NO_DXT5nm
	return packednormal.xyz * 2 - 1;
##els
	return UnpackNormalDXT5nm(packednormal);
##endif
}
```

在世界空间计算光照：计算切线空间→世界空间的变换矩阵，在frag中读取法线贴图然后将切线空间的法线信息转换到世界空间

法线的空间变换（法线空间→世界空间）：

```glsl
struct a2v
{
	//...
	float3 normal : NORMAL;
	float4 tangent : TANGENT;
}

struct v2f
{
	//...
	float3 worldNormalDir : TEXCOORD1;
	float4 worldTangentDir : TEXCOORD2;
	float3 worldbiTangentDir : TEXCOORD3;
}

v2f vert(a2v v)
{
	//...
	//得出世界空间下的法线、切线，并计算出副切线
	o.worldNormalDir = UnityObjectToWorldNormal(v.normal);
	o.worldTangentDir = normalize(mul(unity_ObjectToWorld, float4( v.tangent.xyz, 0.0)).xyz);
	o.worldbiTangentDir = normalize(cross(o.worldNormalDir, o.worldTangentDir) * v.tangent.w);
}

fixed4 frag(v2f i) : SV_Target
{
	//得到变换矩阵（世界空间下的切线、副切线、法线）
	float3x3 tangentTransform = float3x3(i.worldTangentDir, i.worldbiTangentDir, i.worldNormalDir);
	//读取法线贴图并将其转换到世界空间下
	float3 normalLocal = UnpackNormal(tex2D(_NormalMap, TRANSFORM_TEX(i.uv0, _NormalMap)));
	float3 normalWorld = normalize(mul(normalLocal.rgb, tangentTransform));
}
```

## CUBE Map

做反射效果、环境光效果时会用到CUBE Map

用反射向量来读取CUBE Map

```glsl
//获取反射向量
float3 worldRefl = normalize(reflect(-viewDir, finiNormal));
//基于反射向量去读取反射效果的CUBE Map
float4 refCol = texCUBElod(_Cubemap, float4(worldRefl.rgb, (255-_Gloss)*8/(255)))*_EnvScale;
//参数解释：
//_Cubemap:存储反射信息的CubeMap
//_Gloss:光泽度
//_EnvScale:环境影响程度，即反射程度
//函数解释：
//texCUBElod如果传入float4，则最后一位是用来表示lod等级的（mipmap level）
//此处mipmap level是通过_Gloss光泽度来计算的，在Unity中如果勾选了贴图的generate1 mipmap，Unity会自动为这张帖图生成8个等级的mipmap
//用_Gloss计算mipmap level：如果一个物体_Gloss越高，说明物体表面越光滑，越能反射环境，mipmap level等级越低
```

## IBL

IBL：image based lighting，基于图像的光照，把一张图当成光源来加入物体的光照计算中

通常会用到texCUBElod来模拟PBR中粗糙度的表现（同上方CUBE Map中对_Gloss的解释，只不过换成了对光的反射率/显示出的亮度）

## 处理漫反射暗面死黑的trick

通过插值计算漫反射（暗部以环境光作为光源，亮部以真是光源作为光源）

```glsl
float3 diffuse = lerp(ambient.rgb*_Diffuse.rgb*MainTex.rgb, _LightColor.rgb*_Diffuse.rgb*MainTex.rgb, NdotL);
```

## 有金属质感，同时兼具非金属质感的trick

#### 高光计算的trick

模拟PBR效果：通过gloss的值对specular和diffuse进行插值，当Gloss很高（金属质感）时高光颜色与漫反射无关，当Gloss很低时（非金属质感）高光颜色会被漫反射影响

```glsl
float3 specular = _LightColor0.rgb * _SpecularColor.rgb * pow(NdotH, _Gloss);
specular = lerp(diffuse * specular, specular, _Gloss/255);
```

#### 环境映射的trick

和高光计算同理

```glsl
reflColor = lerp(diffuse * reflColor, reflColor, _Gloss/255);
```

###### 很不标准，不建议用，只是一种强行对PBR的仿制

