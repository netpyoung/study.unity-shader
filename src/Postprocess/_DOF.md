# Depth of Field

- DOF: Depth Of Field
- CoC: Circle of Confusion
- Bokeh

- 초점을 맞춘 이 위치의 물체를 제외하고는 전경과 배경이 모두 흐려짐.
- 흐릿함의 정도는 초점에서 멀어질수록(비선형적으로) 커짐.

|           |               |
| --------- | ------------- |
| Coc       | depth texture |
| soft edge | blur          |

## CoC(Circle of Confusion)

착란원

``` txt
// SIGGRAPH2018 A Life of a Bokeh

A         = Apertune diameter         // 센서크기
F         = Focal length              // 피사체와의 거리
P         = Focus distance            // 초점거리
MaxBgdCoc = (A * F)/(P - F)           // 배경의 최대 착란원의 반경
Coc(z)    = (1 - (P / z)) * MaxBgdCoc // 혼란원(확산원)
```

``` hlsl
float z = LinearEyeDepth(SAMPLE_DEPTH_TEXTURE(_CameraDepthTexture,i.uv));
float CoC = (1 - (_FocusDistance / z));
CoC = clamp(CoC, -1, 1) * _MaxBgdCoc;
```

## REF

- <https://catlikecoding.com/unity/tutorials/advanced-rendering/depth-of-field/>
- [slideshare.net/youpyo/d2-depth-of-field](https://www.slideshare.net/youpyo/d2-depth-of-field)
- <https://blog.naver.com/eryners/220152909470>
- ShaderX3 Advanced Rendering with DirectX and OpenGL
  - 4.4 Improved Depth-of-Field Rendering
- <https://qiita.com/UWATechnology/items/2742e5726f3b8a01796d>
- SIGGRAPH2018 A Life of a Bokeh
  - [A Life Of A Bokeh Advances in real-time rendering SIGGRAPH 2018 UE4](https://www.youtube.com/watch?v=wIm25R5Z9lc)
- <https://www.slideshare.net/ohyecloudy/gpu-gems3-chapter-28-practical-post-process-depth-of-field>
