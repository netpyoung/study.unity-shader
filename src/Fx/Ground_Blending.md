# Ground Blending

Blending meshes with terrain

## 사전 캡쳐 이용

- 카메라를 상공에 두고 지형만 G버퍼와 Y위치를 저장하여, 오브젝트가 지형과 블랜딩시 해당 정보를 이용하여 블랜딩한다.


|                         | builtin | urp           |
| ----------------------- | ------- | ------------- |
| Camera.RenderWithShader | Yes     | Not supported |
| Camera.AddCommandBuffer | Yes     | Not supported |
| Camera.Render           | Yes     | Not supported |
| Light.AddCommandBuffer  | Yes     | Not supported |
| OnPreCull               | Yes     | Not supported |
| OnPreRender             | Yes     | Not supported |
| OnPostRender            | Yes     | Not supported |
| OnRenderImage           | Yes     | Not supported |

RenderRequest api that Robin mentioned also landed in 2022.2 

- https://forum.unity.com/threads/replacement-for-camera-render-camera-renderwithshader.845854/

``` cs
RenderPipeline.StandardRequest request = new RenderPipeline.StandardRequest();
request.destination = aoRT;
camera.GetUniversalAdditionalCameraData().SetRenderer(k_AOBakeRendererID);
RenderPipeline.SubmitRenderRequest(camera, request);
```

## VT 이용

- 언리얼에서는 
  - 런타임 버추얼 텍스처 볼륨(Runtime Virtual Texture Volume)

## Ref

- GPU Pro 1 : Part III : 4 Virtual Texture Mapping 101
- GDC 2008: Sean Barrett
  - https://www.youtube.com/watch?v=MejJL87yNgI
- GDC 2015: Ka Chen : Adaptive Virtual Texture Rendering in Far Cry 4
  - https://www.youtube.com/watch?v=SVPMhGteeuE
- https://www.shadertoy.com/view/XdXGW8
- [[Unity - URP] Ground Object Blending](https://youtu.be/Q2o70xBDe_U?si=5p10Ar-2n1yY8WmK)
  - <https://bitbucket.org/ciitt/ground-object-blending/src/master/>


SDF 이용
 - https://polycount.com/discussion/181140/unreal-4-terrain-blending-tool-inspired-by-star-wars-battlefront

지형과 오브제젝트 교차점 검사
  - 언리얼인 경우 DistanceToNearestSurface
    - 언리얼인 경우 SDF를 통해 distance를 저장함.
버텍스 페인트 마스크
