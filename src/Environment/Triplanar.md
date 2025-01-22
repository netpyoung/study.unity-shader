# Triplanar

세 개의 평면(tri-plane)

- 일반 스케일하면 이상하게 보이는 경우 (ex 바위/땅)
  - 월드좌표기반 uv를 사용하면 여러 오브젝트들이 있을때 자연스럽게 스며듬
- 3번 샘플링 해야함.

``` hlsl
// for Color
float3 vec3_Weight = pow(abs(normalWS), _Sharpness);
float weightSum = dot(vec3_Weight, 1.0);
vec3_Weight /= weightSum;

float3 uv = positionWS * _Scale;
float4 color_Side  = SAMPLE_TEXTURE2D(_TriplanarColorTex, sampler_TriplanarColorTex, uv.zy); // side
float4 color_Top   = SAMPLE_TEXTURE2D(_TriplanarColorTex, sampler_TriplanarColorTex, uv.xz); // top
float4 color_Front = SAMPLE_TEXTURE2D(_TriplanarColorTex, sampler_TriplanarColorTex, uv.xy); // front

float4 triplanar_Color;
triplanar_Color += color_Side  * vec3_Weight.x;
triplanar_Color += color_Top   * vec3_Weight.y;
triplanar_Color += color_Front * vec3_Weight.z;
```

``` hlsl
// for Normal

float3 vec3_Weight = pow(abs(normalWS), _Sharpness);
float weightSum = dot(vec3_Weight, 1.0);
vec3_Weight /= weightSum;

float3 uv = positionWS * _Scale;
float3 normalTS_Side  = UnpackNormal(SAMPLE_TEXTURE2D(_TriplanarNormalTex, sampler_TriplanarNormalTex, uv.zy));
float3 normalTS_Top   = UnpackNormal(SAMPLE_TEXTURE2D(_TriplanarNormalTex, sampler_TriplanarNormalTex, uv.xz));
float3 normalTS_Front = UnpackNormal(SAMPLE_TEXTURE2D(_TriplanarNormalTex, sampler_TriplanarNormalTex, uv.xy));
normalTS_Side  = float3(normalTS_Side.xy  + normalWS.zy, abs(normalTS_Side.z)  * normalWS.x);
normalTS_Top   = float3(normalTS_Top.xy   + normalWS.xz, abs(normalTS_Top.z)   * normalWS.y);
normalTS_Front = float3(normalTS_Front.xy + normalWS.xy, abs(normalTS_Front.z) * normalWS.z);

float3x3 matrix_TBN = float3x3(IN.WS_Tangent, IN.WS_BiTangent, IN.WS_Normal);
float3 triplanar_NormalWS;
triplanar_NormalWS += normalTS_Side.zyx  * vec3_Weight.x;
triplanar_NormalWS += normalTS_Top.xzy   * vec3_Weight.y;
triplanar_NormalWS += normalTS_Front.xyz * vec3_Weight.z;
triplanar_NormalWS = normalize(triplanar_NormalWS);
float3 triplanar_NormalTS = TransformWorldToTangent(triplanar_NormalWS, matrix_TBN);
```


- https://developer.nvidia.com/gpugems/gpugems3/part-i-geometry/chapter-1-generating-complex-procedural-terrains-using-gpu
  - 1.5 Texturing and Shading


- https://www.ronja-tutorials.com/post/010-triplanar-mapping/