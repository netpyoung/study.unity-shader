# Lighitng Model - PBR

물리기반

## Cook Torrance - 쿡토렌스

- 1982 - Robert L.Cook & Kenneth E. Torrance - A Reflectance Model For Computer Graphics
- 미세면이론
- 거친표면 specular 초점

## Ward - 알드

- 1992 - Gregory J. Ward - Measuring and modeling anisotropic reflection
- 경험적 데이터 기반, 거의 사용되지 않음.

## Oren-Nayar - 오렌네이어

- 1994 - Michael Oren & Shree K. Nayar - Generalization of Lambert’s Reflectance Model
- 거친포면 diffuse 초점

``` hlsl
half NdotL = max(0.0, dot(N, L));
half NdotV = max(0.0, dot(N, V));
half VdotL = max(0.0, dot(V, L));

half s = VdotL - NdotL * NdotV;
half t = lerp(1.0, max(NdotL, NdotV), step(0.0, s));

half3 A = 1.0 + _OrenNayarAlbedo * (_OrenNayarAlbedo / (_OrenNayarSigma + 0.13) + 0.5 / (_OrenNayarSigma + 0.33));
half3 B = 0.45 * _OrenNayarSigma / (_OrenNayarSigma + 0.09);

half3 diffuse = _OrenNayarAlbedo * max(0.0, NdotL) * (A + B * s / t) / 3.14159265;
```

## Modified Phong - 모디파이드 퐁

- Lafortune and Willems (1994)

``` hlsl
half norm = (shininess + 2.0) / (2.0 * PI);

half3 R = reflect(-L, N);
half3 VdotR = max(0.0, dot(V, R));

half3 specular = norm * pow(VdotR, shininess);
```

## Ashikhmin Shirley - 어크먼 셜리

- 2000 - Michael Ashikhmin & Peter Shirley - An Anisotropic Phong BRDF Model
- 퐁 스펙큘러

## Fakey Oren-Nayar -  최적화 오렌네이어

- 2011 - [pope - Rendering Tech of Space Marine](https://www.slideshare.net/blindrenderer/rendering-tech-of-space-marinekgc-2011)
  - <https://kblog.popekim.com/2011/11/blog-post_16.html>

``` hlsl
half OrenNayar_Fakey(half3 N, half3 L, half3 V, half roughness)
{
    half LdotN = dot(L, N);
    half VdotN = dot(V, N);

    half result = saturate(LdotN);
    half soft_rim = saturate(1 - VdotN / 2);

    const half FAKEY_MAGIC = 0.62;
    half fakey = pow(1 - result * soft_rim, 2);
    fakey = FAKEY_MAGIC - fakey * FAKEY_MAGIC;
    return lerp(result, fakey, roughness);
}
```

## Disney - 디즈니

- SIGGRAPH 2012 - Brent Burley - Physically Based Shading at Disney
- 여러 파라미터

## Ref

- <https://www.cnblogs.com/timlly/p/10631718.html>
- <https://www.jordanstevenstechart.com/physically-based-rendering>
