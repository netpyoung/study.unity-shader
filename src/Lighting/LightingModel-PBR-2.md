# Lighitng Model - PBR 2


ImageBasedLighting.hlsl PBRjcnt IBL관련된 부분(GGX, Anisotropy, ImportanceSample 등)

## PBR에 대해 아는지 물어보려면?

IBL: ImageBasedLighting
GGX: Ground Glass unknown
roughness와 smoothness 관계 


|      |                                                   |                                                |
| ---- | ------------------------------------------------- | ---------------------------------------------- |
| BRDF | Bidirectional reflectance distribution function   | 는 빛이 어떤 방향으로 반사가 되는지            |
| BTDF | Bidirectional transmittance distribution function | 는 빛이 어떤 방향으로 투과가 되는지            |
| BSDF | Bidirectional scattering distribution function    | 이 둘을 합쳐 빛이 재질과 어떻게 상호작용하는지 |



The Blinn-Phong Normalization Zoo: http://www.thetenthplanet.de/archives/255

알베도 (albedo)
 - 입사광에 대한 반사광의 비율입니다.
 - 알베도는 물체의 밝기를 나타내는 것이 아니라 외부의 빛에 대한 것입니다. 


| Specular        |                           |                                   |
| --------------- | ------------------------- | --------------------------------- |
| D(Distribution) | 미세면 분포 함수          | NDF(Normal Distribution Function) |
| G(Geometry)     | 미세면 그림자 마스킹 함수 | 기하(Geometry)와 관련             |
| F(Fresnel)      | 프레넬 함수               |                                   |

## TODO
공식 인덱스만들기

|          |                            |                |
| -------- | -------------------------- | -------------- |
| Diffuse  |                            |                |
|          | 1760 Lambert               |                |
|          | 1994 Oren&Nayar            |                |
|          | 2011 Pope Fakey Oren-Nayar |                |
|          | 2012 Burley                |                |
| Specular |                            |                |
| - D      | 0000 Blinn-Phong           |                |
| - D      | 0000 Beckmann              |                |
| - D      | 1975 GGX(Trowbridge&Reitz) |                |
| - D      | 0000 GGX Anisotropic       |                |
| - G      | 1999 Neumann               |                |
| - G      | 2001 Kelemen               |                |
| - G      | 1967 Smith                 |                |
| - G      | 0000 GGX                   |                |
| - G      | 1994 Schlick               |                |
| - G      | 0000 Schlick-Beckmann      |                |
| - G      | 0000 Schlick-GGX           |                |
| - G      | 2014 Heitz                 |                |
| - F      | 1982 Cook & Torrance       |                |
| - F      | 1994 Schlick               |                |
| - F      | 2012 Schlick SG            | 싸니까 잘 사용 |

### GGX(Ground Glass unknown)

Cook-Torrance
  Rs = ((F * D * G) / (pi * NdotV * NdotL))

Walter
  Rs = ((F * D * G) / (4 * NdotV * NdotL))

a = roughness
a2 = a * a

### D(Distribution) - microsurface Distribution

``` hlsl
Blinn-Phong

float D_Blinn(float a2, float NoH)
{
	float n = 2 / a2 - 2;
	return (n + 2) / (2 * PI) * ClampPow(NoH, n);
}
float ClampPow(float X, float Y)
{
	return pow(max(abs(X), 0.000001f), Y);
}

Beckmann
float D_Beckmann(float a2, float NoH)
{
	float NoH2 = NoH * NoH;
	return exp((NoH2 - 1) / (a2 * NoH2)) / (PI * a2 * NoH2 * NoH2);
}

GGX (Trowbridge-Reitz)
float D_GGX(float a2, float NoH)
{
	float d = (NoH * a2 - NoH) * NoH + 1;
	return a2 / (PI * d * d);
}

GGX Anisotropic
float D_GGXaniso(float ax, float ay, float NoH, float3 H, float3 X, float3 Y)
{
	float XoH = dot(X, H);
	float YoH = dot(Y, H);
	float d = XoH * XoH / (ax*ax) + YoH * YoH / (ay*ay) + NoH * NoH;
	return 1 / (PI * ax*ay * d*d);
}
```

### F(Fresnel)

``` hlsl
// F0 : Fresnel Reflectance at 0 Degrees, 일반적으로 0.02 ~ 0.05
// https://substance3d.adobe.com/tutorials/courses/the-pbr-guide-part-1

cosA = HdotV

// Schlick 1994
Schlick(H, V, F0)    = F0 + (1.0 − F0) * pow((1.0 − (HdotV)), 5);

// Schlick SG 2012
Schlick_SG(H, V, F0) = F0 + (1.0 - F0) * Pow2((-5.55473 * VdotH - 6.98316) * VdotH);

// Cook-Torrance
```


### G(Geometry) - Geometrical Attenuation

``` hlsl
// Implicit GImplicit(l,v,h)=(NdotL)(NdotV)
// Cook-Torrance
// GGX
// Schlick-Beckmann
// Schlick-GGX
// http://graphicrants.blogspot.com/2013/08/specular-brdf-reference.html

// [Neumann et al. 1999, "Compact metallic reflectance models"]
float Vis_Neumann(float NoV, float NoL)
{
	return 1 / (4 * max(NoL, NoV));
}

// [Kelemen 2001, "A microfacet based coupled specular-matte brdf model with importance sampling"]
float Vis_Kelemen(float VoH)
{
	return rcp(4 * VoH * VoH + 1e-5);
}

// Smith term for GGX
// [Smith 1967, "Geometrical shadowing of a random rough surface"]
float Vis_Smith(float a2, float NoV, float NoL)
{
	float Vis_SmithV = NoV + sqrt(NoV * (NoV - NoV * a2) + a2);
	float Vis_SmithL = NoL + sqrt(NoL * (NoL - NoL * a2) + a2);
	return rcp(Vis_SmithV * Vis_SmithL);
}

// Tuned to match behavior of Vis_Smith
// [Schlick 1994, "An Inexpensive BRDF Model for Physically-Based Rendering"]
float Vis_Schlick(float a2, float NoV, float NoL)
{
	float k = sqrt(a2) * 0.5; // 0.5 ~= 2 / pi
	float Vis_SchlickV = NoV * (1 - k) + k;
	float Vis_SchlickL = NoL * (1 - k) + k;
	return 0.25 / (Vis_SchlickV * Vis_SchlickL);
}

// Appoximation of joint Smith term for GGX
// [Heitz 2014, "Understanding the Masking-Shadowing Function in Microfacet-Based BRDFs"]
float Vis_SmithJointApprox(float a2, float NoV, float NoL)
{
	float a = sqrt(a2);
	float Vis_SmithV = NoL * (NoV * (1 - a) + a);
	float Vis_SmithL = NoV * (NoL * (1 - a) + a);
	return 0.5 * rcp(Vis_SmithV + Vis_SmithL);
}
```

## Microfacet BRDF

- 1963 - Petr Beckmann, André Spizzichino
  - The Scattering of Electromagnetic Waves from Rough Surfaces
  - `D(Distribution)`
- 1967 - B. Smith
  - Geometrical shadowing of a random rough surface
  - `G(Geometry)`
- 1975 - T. S. Trowbridge & K. P. Reitz
  - Average irregularity representation of a rough surface for ray reflection
  - `D(Distribution)`
- 1980 Whitted, T. 
  - An improved illumination model for shaded display. Communications of the ACM 23 (6), 343–49.
  - 전체 조명 효과에 레이 트레이싱을 사용하는 아이디어를 소개
- 1982 - Robert L.Cook & Kenneth E. Torrance
  - A Reflectance Model For Computer Graphics
  - `Cook` & `Torrance`
  - 미세면이론. 금속 표면 렌더링
  ``` hlsl
  // Cook&Torrance
  Rs = ((F * D * G) /  (pi * NdotV * NdotL))
  ```
- 1994 - Oren Nayar
  - Generalization of Lambert’s Reflectance Model
- 1994 - Christophe Schlick
  - An Inexpensive BRDF Model for Physically-based Rendering
  - Schlick’s approximation
  - `F(Fresnel)`
- 1997Peter Shirley
  - A practitioners’ Assessment of Light Reflection Models 
- 2003 - Mitsubishi Electric Research Laboratories(MERL) - MERL BRDF Database
- 2007 - Bruce Walter & Stephen R. Marschner & Hongsong Li & Kenneth E. Torrance
  - Microfacet Models for Refraction through Rough Surfaces
  - `GGX` 용어 사용
    - `We also introduce a new microfacet distribution, which we call GGX, that provides a closer match for some of oursurfaces than the standard Beckmann distribution function.`
  - pi보다 4로 나누는게 에너지 보존 법칙에 더 적합하다 주장.
  ``` hlsl
  // pi => 4
  Rs = ((F * D * G) / (4 * NdotV * NdotL))
  ```
- 2011 Pope
  - Fakey Oren Nayer
  - https://kblog.popekim.com/2011/11/blog-post_16.html
- 2012 - Brent Burley
  - Physically Based Shading at Disney
  - Disney "principled" BRDF
- 2012 - Sébastien 
  - https://seblagarde.wordpress.com/2012/06/03/spherical-gaussien-approximation-for-blinn-phong-phong-and-fresnel/
  - Approximate Fresnel with SG(Spherical Gaussian)
  - `F(Fresnel)`
- 2014 - Gulbrandsen
  - 프레넬 
  - RGB 렌더링에서 근사화되었으며 Schlick의 근사식보다 정확하지 않았습니다
- 2014 - Eric Heitz
  - Understanding the Masking-Shadowing Function in Microfacet-based BRDFs 
  - `G(Geometry)` Smith Joint Approx
- 2015 - Disney
  - Disney BSDF
- 2019 - Naty Hoffman
  - 에 의해 그들보다 정확한 F82 파라미터(82도에서의 반사율)의 도입이 제안되었다. 그러나, F82 파라미터는 에너지 보존에 문제가 존재 했다
- 2021
  - 「Adobe Standard Material」가 등장해, 거기에는 그 개량판인 F82-tint(82도에 있어서의 색조)가 도입되었다 [18] .

G2(L,V,m): % visible in 2 directions
G1(V,m): % visible in just 1 direction
G1(L,m): % visible in just 1 direction

## 사례

- IGC2017 검은사막 속 3D Rendering
  - https://www.inven.co.kr/webzine/news/?news=184847

| 검은사막 |                         |           |
| -------- | ----------------------- | --------- |
| Diffuse  |                         |           |
|          | 1994 Oren&Nayar         | optmized? |
| Specular |                         |           |
| - D      | 1975 Trowbridge & Reitz |           |
| - G      | 1982 Cook & Torrance    |           |
| - F      | 2012 Schlick SG         |           |
|          |                         |           |
| Upsize   | Bicubic Upsizing        |           |

- NDC19 모바일에서 사용가능한 유니티 커스텀 섭스턴스 PBR 셰이더 만들기
  - https://www.slideshare.net/dongminpark71/ndc19-pbr-143928930
  - https://www.slideshare.net/dongminpark71/ndc19-pbr-143928930/72
  - https://github.com/TwoTailsGames/Unity-Built-in-Shaders
    - https://github.com/TwoTailsGames/Unity-Built-in-Shaders/blob/master/CGIncludes/UnityStandardUtils.cginc
  - https://github.com/whisperlin/utils/blob/master/unity/UE4PBR/Assets/ue4.cginc
  - https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf

| 서브스텐스 |                             |                         |
| ---------- | --------------------------- | ----------------------- |
| Diffuse    |                             |                         |
|            | 1760 Lambert                |                         |
| Specular   |                             |                         |
| - D        | 1975 GGX (Trowbridge&Reitz) | alpha = Pow2(Roughness) |
| - G        | 1967 Smith                  |                         |
| - F        | 2012 Schlick SG             |                         |

| 텍스쳐 채널 |           |
| ----------- | --------- |
| 1: rgb      | Albedo    |
| 1: a        | Roughness |
| 2: r        | AO        |
| 2: g        | Normal_X  |
| 2: b        | Metallic  |
| 2: a        | Normal_Y  |



| Builtin           |                                                                                |
| ----------------- | ------------------------------------------------------------------------------ |
| BRDF1_Unity_PBS() |                                                                                |
| - D               | GGX or Normalized BlinnPhong (UNITY_BRDF_GGX가 정의되었는지 여부에 따라 다름)  |
| - G               | Smith Joint or Smith Beckmann (UNITY_BRDF_GGX가 정의되었는지 여부에 따라 다름) |
| - F               | Schlick approximation                                                          |
| BRDF2_Unity_PBS() |                                                                                |
| - D               | BlinnPhong 또는 [수정된] GGX                                                   |
| - G               | 수정된 Kelemen 및 Szirmay-Kalos                                                |
| - F               | 1/LdotH로 근사한 프레넬                                                        |
| BRDF3_Unity_PBS() |                                                                                |
| - D               | RDF 형식의 정규화된 BlinnPhong                                                 |
| - G               | RDF 형식의 정규화된 BlinnPhong                                                 |
| - F               | 프레넬 항 없음                                                                 |



## REF

- https://ja.wikipedia.org/wiki/物理ベースシェーディング
- pbr-book
  - https://www.pbr-book.org/3ed-2018/Introduction/A_Brief_History_of_Physically_Based_Rendering
  - https://www.pbr-book.org/3ed-2018/Introduction/Further_Reading
- https://jcgt.org/published/0007/04/01/paper.pdf
  - Sampling the GGX Distribution of Visible Normals - Eric Heitz - Unity Technologies
- https://github.com/netpyoung/bs.physically_based_shader_develop_for_unity

https://catlikecoding.com/unity/tutorials/rendering/part-4/
https://catlikecoding.com/unity/tutorials/rendering/part-8/

- https://light11.hatenadiary.com/entry/2020/03/05/220957

[BRDFモデルの変遷] https://www.slideshare.net/teppeikurita/brdf-196782059

http://boundingboxsoftware.com/materialize/index.php
https://qiita.com/_Pheema_/items/f1ffb2e38cc766e6e668