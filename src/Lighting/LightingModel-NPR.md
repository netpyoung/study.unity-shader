# Lighitng Model - NPR

비 물리기반

### Lambert - 람버트

- Johann Heinrich Lambert
- 1760 - Photometria

``` hlsl
half NdotL = max(0.0, dot(N, L));
half diffuse = NdotL;
```

### Minnaert - 미네르트

- 1954 - Marcel Minnaert

- 달표면 반사를 표현하기 위해 고안됨. moon shader라 불리기도 함
- <https://blog.naver.com/canny708/221551395976>

``` hlsl
half NdotL = max(0.0, dot(N, L));
half NdotV = max(0.0, dot(N, V));
half diffuse = NdotL * pow(NdotL * NdotV, _MinnaertDarkness);
```

### Phong - 퐁

- 1973 - Bui Tuong Phong

``` hlsl
half3 R = reflect(-L, N);
half RdotV = max(0.0f, dot(R, V));
half specular = pow(RdotV, _SpecularPower) * _SpecularNormFactor;
```

### Blinn Phong - 블린 퐁

- 1977 - Jim Blinn

``` hlsl
half3 H = normalize(V + L); 
half NdotH = max(0.0, dot(N, H));

half specular = pow(NdotH ,_SpecularPower) * _SpecularNormFactor;
```

### Strauss - 스트라우스

- 1990 - Paul Strauss
- <https://blog.naver.com/sorkelf/401550597481>

### Gooch - 구치

- 1998 - Gooch
- [SIGGRAPH1998 - A Non-Photorealistic Lighting Model For Automatic Technical Illustration](https://users.cs.northwestern.edu/~ago820/SIG98/abstract.html)
- [GDC2008 - Stylization with a Purpose: The Illustrative World of TEAM FORTRESS 2](https://www.gdcvault.com/play/279/Stylization-with-a-Purpose-The)
- 따뜻함과 차가움 영역을 나누어 표시

### Half Lambert & Wrapped Lambert - 하프 람버트 & 와프드 람버트

- 2004 Half-Life2 - Valve
- [SIGGRAPH2006 - Shading In Valve's Source Engine](https://steamcdn-a.akamaihd.net/apps/valve/2006/SIGGRAPH06_Course_ShadingInValvesSourceEngine.pdf)

``` hlsl
// half lambert
half NdotL = max(0.0, dot(N, L));
half diffuse = pow(NdotL * 0.5 + 0.5, 2);

// wrapped lambert
half diffuse = pow(NdotL * wrapValue + (1.0 - wrapValue), 2);
half diffuse = max(0.0, (NdotL + _wrapped) / (1.0 - _wrapped));
```

``` hlsl
// ref: https://blog.naver.com/eryners/220144182154
// Harf Lambert사용시 명암 차이가 너무 없어져서 무게감이 없어보인다.
half diffuse = ​pow((dot(N, L) * 0.5) + 0.5, 4)  // Half Lambert + Pow
```

``` hlsl
half diffuse = max(0, ((dot(L, N) + warp) / (1 + wrap + wrap^2)) ^ (1 + wrap));
```

## LUT

- Look Up Texture : 룩업텍스쳐
- Ramp Texture라고도 함
  - Ramp : 증감. 경사(gradient)

### Lake

- 2000 - Lake
- [Stylized Rendering Techniques For Scalable Real-Time 3D Animation](http://www.markmark.net/npar/npar2000_lake_et_al.pdf)
- 룩업텍스쳐 사용
  - NdotL - LUT Texture(1D)

### BARLA

- 2006 - BARLA
- [X-Toon: An extended toon shader - Pascal Barla, Joëlle Thollot, Lee Markosian](https://maverick.inria.fr/Publications/2006/BTM06a/x-toon.pdf)
- 룩업텍스쳐 사용
  - NdotL, Detail - LUT Texture(2D)
