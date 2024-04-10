# Shadow 2

|                        |     |
| ---------------------- | --- |
| TRANSFER_SHADOW        |     |
| TRANSFER_SHADOW_CASTER |     |

## 바닥그림자

| 이름         | 부하     | 형상 | 자기그림자 | 특징             |
| ------------ | -------- | ---- | ---------- | ---------------- |
| 원형         | 가장적음 | 평면 | X          | 원형텍스쳐       |
| 평면 투영    | 적음     | 평면 | X          | 평면에만         |
| 투영 텍스쳐  | 적당     | 지형 | X          | 물체 하나만      |
| 우선순위버퍼 | 큼       | 지형 | X          | 자기 그림자 없음 |
| 스텐실       | 큼       | 지형 | O          | 정점 수가 많음   |
| 깊이버퍼     | 큼       | 지형 | O          | 부분적 깜빡임    |

### 원형(circle) 그림자

- 그냥 원형 텍스쳐로

### 평면 투영 (Planar Projected) 그림자

- 그림자용 모델 따로 만들어서 셰이더 연산 절약.
- 캡슐로 몸통이랑 다리랑만 만들어서 대략적인 계산
  - [基于近似演算的Capsule Shadow（胶囊体阴影）](http://walkingfat.com/%e5%9f%ba%e4%ba%8e%e8%bf%91%e4%bc%bc%e6%bc%94%e7%ae%97%e7%9a%84capsule-shadow%ef%bc%88%e8%83%b6%e5%9b%8a%e4%bd%93%e9%98%b4%e5%bd%b1%ef%bc%89/)

### RenderTexture이용

- Shadow용 모델
  - LOD
- RenderTexture

### 스텐실

``` hlsl
float4 vPosWorld = mul( _Object2World, v.vertex);
float4 lightDirection = -normalize(_WorldSpaceLightPos0); 
float opposite = vPosWorld.y - _PlaneHeight;
float cosTheta = -lightDirection.y;	// = lightDirection dot (0,-1,0)
float hypotenuse = opposite / cosTheta;
float3 vPos = vPosWorld.xyz + ( lightDirection * hypotenuse );
o.pos = mul (UNITY_MATRIX_VP, float4(vPos.x, _PlaneHeight, vPos.z ,1));  

// 그림자 덧 방지
Stencil
{
    Ref 0
    Comp Equal
    Pass IncrWrap
    ZFail Keep
}
```

## 그림자맵 - 깊이버퍼

- 광원을 기준으로 물체의 상대적 거리(0 ~ 1)를 이미지로 저장.(0은 광원의 위치)
- <https://github.com/netpyoung/bs.introduction-to-shader-programming/blob/master/note/ch10.md>

## PSM(Perspective Shadow Map)

- <http://www-sop.inria.fr/reves/Marc.Stamminger/psm/>
- <http://x66vx.egloos.com/3808794>

## Ref

- [게임 개발 포에버: 실시간 그림자를 싸게 그리자! 평면상의 그림자 ( Planar Shadow)](https://gamedevforever.com/326)
- [[1023 박민수] 깊이_버퍼_그림자_1](https://www.slideshare.net/MoonLightMS/1023-1)
- [타카시 이마기레 - DirectX 9 셰이더 프로그래밍](https://www.hanbit.co.kr/store/books/look.php?p_code=B9447539340)
- <https://www.slideshare.net/slideshow/ss-13541521/13541521>




----------



Projected Shadow Mapping
광원방향 투영 => 텍스쳐 맵핑
오브젝트 단위임으로, 오브젝트 자신에는 그림자가 안드리워짐

Stencil Shadow Volume
광원기준 차폐여부 판별

- 플로우
  - 씬 깊이정보
  - 그림자 볼륨( 오브젝트를 광원 방향으로 )
  - 스텐실 버퍼(씬 깊이정보와 그림자 볼륨 비교 하여 그림자인지 아닌지)
  - 그림자 그리기(스텐실 참고)

Depth buffer Shadow
그림자맵 - 광원 기준으로 오브젝트들과의 거리.
그림자맵 정밀도 == 그림자맵의 해상도

PSM: Perspective

카메라와의 거리를 참고하여(ViewxProjection), 가까운 오브젝트의 해상도를 높임. 
광원기준 그림자맵

LSPSM: Light Space Perspective


Cascaded LSPSM
계층. 근거리/중거리/장거리 등 여러단계로 나눔

Soft Shadow
계단현상 뭉개기. Post Processing

PCF: percentage-closer filtering
안티얼라이싱

- 방식
  - Shadow 비율 = 픽셀의 Depth 값, Shadow map의 일정 영역의 Depth 값을 비교

VSM: Variance Shadow Map
Variance: 분산
ue3.0 사용
그림자맵(오브젝트와 광원의 거리, 그 제곱값)
그림자가 생길 확률계산하여 그림자 생성
  - 체비쇼프(Pafnuty Chebyshev) 부등식등
 
단점:  Light Bleeding(빛이 그림자 내로 새어들어감)

## Ref

Tutorial 42:
Percentage Closer Filtering
https://ogldev.org/www/tutorial42/tutorial42.html


Tutorial 49:
Cascaded Shadow Mapping
https://ogldev.org/www/tutorial49/tutorial49.html

http://www.opengl-tutorial.org/intermediate-tutorials/tutorial-16-shadow-mapping/

PCF: https://developer.nvidia.com/gpugems/gpugems/part-ii-lighting-and-shadows/chapter-11-shadow-map-antialiasing
VSM: https://developer.nvidia.com/gpugems/gpugems3/part-ii-light-and-shadows/chapter-8-summed-area-variance-shadow-maps


|                  |                           |               |
| ---------------- | ------------------------- | ------------- |
| Lightmap         | static only. 표면정보     |               |
| LightProve       | support dynamic. 공간정보 | diffuse term  |
| Reflection Probe |                           | specular term |

[[유니티 TIPS] 핵심만 쏙쏙! Baked Lighting 이해하기](https://www.youtube.com/watch?v=J4iVXAYaJfQ)



illuminanace = SAMPLE_TEXTURE2D_LIGHTMAP(lightmapTex, lightmapSampler, LIGHTMAP_EXTRA_ARGS_USE).rgb;

MeshRenderer > Lighting > Receive Global Illumination: Lightmaps
MeshRenderer > Lighting > Receive Global Illumination: Light Proves

| Directional Mode |                            |
| ---------------- | -------------------------- |
| Directional      | 노말정보 포함(메모리 커짐) |
| Non-Directional  |                            |

| LightingMode        |                                             |                                        |
| ------------------- | ------------------------------------------- | -------------------------------------- |
| Baked Indirect      | 간접광 저장 +  그림자 realtime              | 동적 그림자에 GI정보까지 더하니 무거움 |
| Subtractive         | 정적오브젝트 baked + 동적오브젝트 realtime. | 그림자 위에 그림자                     |
| Shadowmask          | 쉐도우 마스크만 따로 저장.                  | 그림자간 경계가 부드러움               |
| Distance Shadowmask | 가까운거리는 realtime. 그 외에는 baked.     | Shadowmask Mode에서 선택               |

요세는 Distance Shadowmask 아니면 완전 realtime으로

- [[유니티 TIPS] 새로워진 Memory Profiler 1.0 활용법 소개](https://www.youtube.com/watch?v=rdspAfOFRJI)
  - 디바이스
  - 빌드별
  - 씬별
  - 2022.2 부터. com.unity.memoryprofile

- https://computergraphics.stackexchange.com/questions/54/when-is-a-compute-shader-more-efficient-than-a-pixel-shader-for-image-filtering
  - In the pixel shader version, the source image is sampled multiple times per pixel.
  - In the compute shader version, each source texel is read only once inside a work group.
- https://tech.spark-creative.co.jp/entry/2021/12/21/121227


Oh Thank you I found option what i want (desaturate: 100). I didn't notice there is filter option.

And I found expression option. It little bit slow when I turn on in my windows machine but work well.

It will be good to game programmer if support to matrix operation per pixel or per group of pixels.

and Save/Load filters.

## example

- per pixel matrix example

``` hlsl
half3x3 sepiaVals = half3x3
(
    0.393, 0.349, 0.272,    // Red
    0.769, 0.686, 0.534,    // Green
    0.189, 0.168, 0.131     // Blue
);

half3 sepiaResult = mul(tex.rgb, sepiaVals);
```

- per group of pixels example

``` dummy
identity_mat_3x3 =
 [[0, 0, 0]
  [0, 1, 0]
  [0, 0, 0]]

origin_image * identity_mat_3x3 => origin_image

sharpen_mat_3x3 =
 [[0, -1, 0]
  [-1, 5, -1]
  [0, -1, 0]]
origin_image * sharpen_mat_3x3 => sharpen_image
```

## Ref

- Luminance
  - https://github.com/Unity-Technologies/FPSSample/blob/6b8b27aca3690de9e46ca3fe5780af4f0eff5faa/Packages/com.unity.postprocessing/PostProcessing/Shaders/Colors.hlsl#L228
- Kernel
  - https://en.wikipedia.org/wiki/Kernel_(image_processing)
- sepia filter
  - https://danielilett.com/2019-05-01-tut1-1-smo-greyscale/
  - https://learn.microsoft.com/en-us/archive/msdn-magazine/2005/january/net-matters-sepia-tone-stringlogicalcomparer-and-more
- sharpen
  - https://datahacker.rs/004-how-to-smooth-and-sharpen-an-image-in-opencv/


   sepia.r = dot(tex.rgb, float3(0.393f, 0.769f, 0.189f));
   sepia.g = dot(tex.rgb, float3(0.349f, 0.686f, 0.168f));
   sepia.b = dot(tex.rgb, float3(0.272f, 0.534f, 0.131f));