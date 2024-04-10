# Color Grading LUT

색 보정(Color Grading)에는 여러 수치 보정 수식들이 들어가게 되는데, 이걸 실시간 계산이 아닌 텍스쳐에 구어 연산 부하를 낮출 수 있다.

- unity: 32x32 가 옆으로 32개 => 1024 × 32
  - [NeutralLdrLut.png](https://github.com/Unity-Technologies/Graphics/blob/master/com.unity.postprocessing/PostProcessing/Textures/LUTs/NeutralLdrLut.png)
  - ldr(Low Dynamic Range)



``` hlsl
// (r,g,b)
// (0,1,0) +----+ (1,1,0)          (0,1,n) +----+ (1,1,n)
//         |    |           ......         |    |        
// (0,0,0) +----+ (1,0,0)          (0,0,n) +----+ (1,0,n)
```



- [http://ttmayrin.tistory.com/34](https://web.archive.org/web/20120213004214/http://ttmayrin.tistory.com/34)

``` hlsl
// 16 기준

float3 CalcLUT2D( sampler InLUT, float3 InColor )
{
    // requires a volume texture 16x16x16 unwrapped in a 2d texture 256x16
    // can be optimized by using a volume texture
    float2 Offset = float2(0.5f / 256.0f, 0.5f / 16.0f);
    float Scale = 15.0f / 16.0f; 

    // Also consider blur value in the blur buffer written by translucency
    float IntB = floor(InColor.b * 14.9999f) / 16.0f;
    float FracB = InColor.b * 15.0f - IntB * 16.0f;

    float U = IntB + InColor.r * Scale / 16.0f;
    float V = InColor.g * Scale;

    float3 RG0 = tex2D( InLUT, Offset + float2(U               , V) ).rgb;
    float3 RG1 = tex2D( InLUT, Offset + float2(U + 1.0f / 16.0f, V) ).rgb;

    return lerp( RG0, RG1, FracB );
}

float3 CalcLUT3D( sampler InLUT, float3 InColor )
{
    return tex3D( InLUT, InColor * 15.f / 16.f + 0.5f / 16.f ).rgb;
}
```

## 색보정 방법

채도/대비/샤픈
마젠타/사이언/그린

- [영화같은 색보정을 위한 가장 중요한 이것](https://youtu.be/ll3HrF4y27w?si=addpL3anqHu0FGep)
- 하이비드 HYVID STUDIO
  - [눈대중으로 색보정 그만! 기준을 갖고 하는 색보정 (feat 피부톤)](https://youtu.be/8V_FoVPR9lg?si=iOobuvMnPO3m1YcP)
  - [매드맥스 DI 작업자 에릭의 헐리웃 색보정 팁](https://youtu.be/H84KGpryPU0?si=4NC9Pudu1jOti1lE)
  - [존윅 4 개봉!! 시선을 사로잡는 존윅의 색보정 비밀! / DI 작업자 질 보그다노비치 색보정 작업방식](https://youtu.be/H_RhJocZWzE?si=SnCBe8kzJHeUjQhF)
- <https://en.wikipedia.org/wiki/Zone_System>

## Ref

- [DirectX Shader LUT 필터 코드 구현](https://nellfamily.tistory.com/51)
- [[Unite Seoul 2019] 최재영 류재성 - 일곱개의 대죄 : "애니메이션의 감성을 그대로＂와 “개발 최적화"](https://youtu.be/0LwlNVS3FJo?t=1087)
- [[GPU Gems 2] Chapter 24. Using Lookup Tables to Accelerate Color Transformations](https://developer.nvidia.com/gpugems/gpugems2/part-iii-high-quality-rendering/chapter-24-using-lookup-tables-accelerate-color)
