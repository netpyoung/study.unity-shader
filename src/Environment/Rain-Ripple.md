# Rain Ripple

빗방울 물결

- 물결 영역(R) : 원형(그라디언트)
- 물결 노말(G,B)
- 물결 시간오프셋(A)
- 물결 패턴 : Sine Wave
- 물결 페이드: 영역 * 패턴
- 물결 왜곡(distortion): 노말 * uv


``` hlsl
float3 ComputeRipple(float2 UV, float CurrentTime, float Weight)
{
    float4 Ripple = tex2D(RippleTexture, UV);
    Ripple.yz = Ripple.yz * 2 - 1; // Decompress perturbation

    float DropFrac = frac(Ripple.w + CurrentTime); // Apply time shift
    float TimeFrac = DropFrac - 1.0f + Ripple.x;
    float DropFactor = saturate(0.2f + Weight * 0.8f - DropFrac);
    float FinalFactor = DropFactor * Ripple.x * sin( clamp(TimeFrac * 9.0f, 0.0f, 3.0f) * PI);

    return float3(Ripple.yz * FinalFactor * 0.35f, 1.0f);
}

// https://seblagarde.wordpress.com/2013/01/03/water-drop-2b-dynamic-rain-and-its-effects/
```



## Ref

- [2017 - [GDC2017] Technical Artist Bootcamp: Shaders 201: Creating Art with Math](https://gdcvault.com/play/1024282/Technical-Artist-Bootcamp-Shaders-201)
  - <https://youtu.be/wfh3iSkNMOQ?si=TV8Hv4N94WCza2n3&t=1207>
1. The CG Tutorial: The Definitive guide to Programmable Real Time Graphics - Randima Fernando
2. Shaders for Game Programmers and Artists - Sebastien St-Laurent
3. The COMPLETE Effect and HLSL Guide -  Sebastien St-Laurent
4. GPU Gems Series edited by Matt Pharr and Randima Fernando
5. Advanced Lighting and Materials with Shaders - Kelly Dempski and Emmannuel Viale

