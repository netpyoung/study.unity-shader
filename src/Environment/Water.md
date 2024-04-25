# 물

- <https://ateliersera.blog.me/220413097549>
- <https://www.ronja-tutorials.com/2018/11/03/river.html>
- <https://alexanderameye.github.io/simple-water.html>
- <https://darkcatgame.tistory.com/30?category=806332>
- <https://github.com/simplestargame/UniversalOceanShaderGraph>
- <https://www.gamedeveloper.com/programming/water-interaction-model-for-boats-in-video-games-part-2>
- <https://www.e2gamedev.com/water-buoyancy>
- [Acerola - Rendering Water With Sine Waves](https://www.youtube.com/watch?v=PH9q0HNBjT4)
- [Acerola - I Tried Simulating The Entire Ocean](https://youtu.be/yPfagLeUa7k?si=n9owb07c8P-rgG8c)
- [Ocean waves simulation with Fast Fourier transform](https://youtu.be/kGEqaX4Y4bQ?si=IyHfueQ_X_CiNIrN)


 
- 위치
  - sin을 이용한 리니어 방식
    - linear wave : y = y - Amplitude * sin(Frequency * x - Phase)
  - noise를 이용한 방식
- 디테일
  - 노말맵
    - 1장 - 단조로움
    - 2장 - 역방향으로 하고 합친다
      - n = n1 + n2 * 0.5 * (0.5, 0.5, 1)
  - 버텍스칼라 가중치


- 주의해야할 점
  - 너무 단순히 만들면 타일링 패턴으로, 부자연스럽게 보임

## 바다/강

- 특징 짚기
  - 물결
  - 거품
    - sea form
      - 오브젝트 상호작용은 깊이맵 이용
    - sea spray
  - 거리 가까울때 투명
  - 거리 멀때 거울처럼 반사
    - cubemap ? Planar Reflection
  - 수면 아래 Caustic
    - 카메라 앞에 커스틱 매쉬를 둬서 처리


## 셰이더 참고 단어

|           |                                               |
| --------- | --------------------------------------------- |
| Amplitude | 웨이브 진폭(amplitude)                        |
| Caustic   | 커스틱. 반사/굴절광이 다른 물체에 맺히는 특성 |

## Ref

- gpugems 1
  - [Chapter 1. Effective Water Simulation from Physical Models](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-1-effective-water-simulation-physical-models)
  - [Chapter 2. Rendering Water Caustics](https://developer.nvidia.com/gpugems/gpugems/part-i-natural-effects/chapter-2-rendering-water-caustics)
  - [Chapter 42. Deformers](https://developer.nvidia.com/gpugems/gpugems/part-vi-beyond-triangles/chapter-42-deformers)
- gpugems 2
  - [Chapter 18. Using Vertex Texture Displacement for Realistic Water Rendering](https://developer.nvidia.com/gpugems/gpugems2/part-ii-shading-lighting-and-shadows/chapter-18-using-vertex-texture-displacement)
    - [번역](https://blog.naver.com/lifeisforu/80026280288)
- Boat Attack
  - https://github.com/Unity-Technologies/BoatAttack
  - [모바일 플랫폼을 위한 URP에서의 물 표현. 모바일 플랫폼을 위한 물 표현 (1/6)](https://www.youtube.com/watch?v=LyiRALUOQqo&list=PL412Ym60h6uvBOpPSP-tcnINt971OD7ZC)
  - 2 Gerstner wave - 카메라를 따라다니게
  - 3 Planar Reflection
  - 4 Caustic - 카메라를 따라다니게
  - 5 Fresnel
- <https://en.wikipedia.org/wiki/Trochoidal_wave>
- [Water Caustics Shader - Unity ](https://www.youtube.com/watch?v=ofLYUlhoxAI)
  - <https://github.com/z4gon/water-caustics-shader-unity>
- Deep-Water Animation and Rendering
  - <https://www.semanticscholar.org/paper/Deep-Water-Animation-and-Rendering-Jensen-Goli%C3%A1%C5%A1/b82acf6a543aff6a3581b1e7ad02efb88b501750>
  - [번역](https://blog.naver.com/lifeisforu/80104510751)
- https://catlikecoding.com/unity/tutorials/flow/waves/


- [[SIGGRAPH2018] The Technical Art of Sea of Thieves](https://www.youtube.com/watch?v=y9BOz2dFZzs)
  - https://history.siggraph.org/learning/the-technical-art-of-sea-of-thieves/


``` hlsl
float3 GerstnerWave (
      float4 wave,
      float3 p,
      inout float3 tangent,
      inout float3 binormal)
{
  float steepness = wave.z;
  float wavelength = wave.w;
 
  float  k = 2 * UNITY_PI / wavelength;
  float  c = sqrt(9.8 / k);
  float2 d = normalize(wave.xy);
  float  f = k * (dot(d, p.xz) - c * _Time.y);
  float  a = steepness / k;
  
  float sinf;
  float cosf;
  sincos(f, sinf, cosf);

  tangent += float3(
    -d.x * d.x * (steepness * sinf),
    d.x * (steepness * cosf),
    -d.x * d.y * (steepness * sinf)
  );
  binormal += float3(
    -d.x * d.y * (steepness * sinf),
    d.y * (steepness * cosf),
    -d.y * d.y * (steepness * sinf)
  );
  
  return float3(
    d.x * (a * cosf),
    a * sinf,
    d.y * (a * cosf)
  );
}
```

퓨리에변환
fourier transform
https://en.wikipedia.org/wiki/Fourier_transform

웨블레트 (Wavelet)
웨이블릿이란 0을 중심으로 증가와 감소를 반복하는 진폭을 수반한 파동 같은 진동을 말한다


phillips spectrum
https://github.com/Scrawk/Phillips-Ocean