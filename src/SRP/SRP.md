# SRP (Scriptable Render Pipeline)

https://docs.unity3d.com/Manual/render-pipelines-feature-comparison.html

## RenderPipelineAsset.asset

- 유니티에서 그래픽스 파이프 라인을 관리한다. 여러 `Renderer`를 가질 수 있다.

``` cs
// 런타임 렌더파이프라인에셋 교체

// Edit> Project Settings> Scriptable Render Pipeline Settings

// 혹은, 이하 스크립트
public RenderPipelineAsset _renderPipelineAsset;

GraphicsSettings.renderPipelineAsset = _renderPipelineAsset;
```

``` cs
// 클래스를 만들어서 사용자 렌더파이프라인에셋 만들기
[CreateAssetMenu(menuName = "Rendering/CustomRenderPipelineAsset")]
public class CustomRenderPipelineAsset : RenderPipelineAsset
{
    protected override RenderPipeline CreatePipeline()
    {
        return new CustomRenderPipeline();
    }
}

public class CustomRenderPipeline : RenderPipeline
{
    protected override void Render(ScriptableRenderContext context, Camera[] cameras);
}
```

## Example

### RenderPipeline

``` hlsl
Pass
{
    Tags
    {
        // LightMode 태그는 라이팅 파이프 라인에서 패스의 역할을 정의.
        "LightMode" = "CustomLightMode"
    }
}

```

``` cs
[CreateAssetMenu(menuName = "Rendering/CustomRenderPipelineAsset")]
public class CustomRenderPipelineAsset : RenderPipelineAsset
{
    protected override RenderPipeline CreatePipeline()
    {
        return new CustomRenderPipeline();
    }
}

// ==========================================================================
public class CustomRenderPipeline : RenderPipeline
{
    CustomRenderer _renderer = new CustomRenderer();

    protected override void Render(ScriptableRenderContext context, Camera[] cameras)
    {
        foreach (Camera cam in cameras)
        {
            _renderer.Render(ref context, cam);
        }
    }
}

// ==========================================================================
public class CustomRenderer
{
    readonly static ShaderTagId unlitShaderTagId = new ShaderTagId("CustomLightMode");

    public void Render(ref ScriptableRenderContext context, Camera cam)
    {
        // ...
        context.Submit();                 // 실행
    }
}
```

```cs
context.SetupCameraProperties(camera); // cmd전에 설정해주자(빠른 지우기)
var cmd = new CommandBuffer();
cmd.ClearRenderTarget
context.ExecuteCommandBuffer(cmd); // enqueue cmd
cmd.Release();
context.Submit();                 // 실행
```

``` cs
var cmd = new CommandBuffer();
cmd.BeginSample(string sampleName); // profiler begin
cmd.EndSample(string sampleName);   // profiler end
```

``` cs
// 컬링
if (!CulllResults.GetCullingParameters(camera, out ScriptableCullingParameters cullingParams))
{
    continue;
}
CullResults cullingResults = context.Cull(ref cullingParams);

SortingSettings sortingSettings = new SortingSettings(cam);
DrawingSettings drawingSettings = new DrawingSettings(unlitShaderTagId, sortingSettings);
FilteringSettings filteringSettings = new FilteringSettings(RenderQueueRange.opaque);

context.DrawRenderers(cullingResults, ref drawingSettings, ref filteringSettings);
context.DrawRenderers             // 렌더링
context.DrawSkybox(camera)        // Skybox
```


``` cs
// cs
var cmd = new CommandBuffer();
cmd.SetGlobalVector("_LightDir", new Vector4(0, 1, 0, 0));
context.ExecuteCommandBuffer(cmd);
cmd.Release();

// shader
CBUFFER_START(_Light) // CommandBuffer에서 전송됨
float4 _LightDir;
CBUFFER_END
```

- CBUFFER_START(UnityPerMaterial) // 메터리얼별
- CBUFFER_START(UnityPerDraw)     // draw별
- <https://blogs.unity3d.com/kr/2019/02/28/srp-batcher-speed-up-your-rendering/>

``` cs
/// Render Texture 사용.

// RenderTarget Id가 필요
int _TmpShaderProperty = Shader.PropertyToID("_TmpShaderProperty");

{
    var cmd = new CommandBuffer();
    // https://docs.unity3d.com/ScriptReference/Rendering.CommandBuffer.GetTemporaryRT.html
    // GetTemporaryRT(int nameID, int width, int height, int depthBuffer, FilterMode filter, RenderTextureFormat format, RenderTextureReadWrite readWrite, int antiAliasing, bool enableRandomWrite);
    // GetTemporaryRT(int nameID, RenderTextureDescriptor desc, FilterMode filter);
    cmd.GetTemporaryRT(_TmpShaderProperty, )
    cmd.SetRenderTarget(RTID);
    cmd.ClearRenderTarget;
    context.ExecuteCommandBuffer(cmd);
    cmd.Release();
}

{
    var cmd = new CommandBuffer();
    cmd.Blit(RTID, BuiltinRenderTextureType.CameraTarget);
    cmd.ReleaseTemporaryRT(TemporaryRTID);
    context.ExecuteCommandBuffer(cmd);
    cmd.Release();
}
```


### ScriptableRenderPass

``` cs

...

CullResults cr = CullResults.Cull(ref cullingParams, context);
InitializeRenderingData(settings, ref cameraData, ref cullResults, out var renderingData);
renderer.Setup(context, ref renderingData); // RenderPass 쌓기.
renderer.Execute(context, ref renderingData);
```

``` cs
public struct RenderingData
{
    public CullingResults cullResults;
    public CameraData cameraData;
    public LightData lightData;
    public ShadowData shadowData;
    public PostProcessingData postProcessingData;
    public bool supportsDynamicBatching;
    public PerObjectData perObjectData;
    public bool postProcessingEnabled;
}
```

``` cs
- ScriptableRenderContext
- ScriptableRenderer (abstract class)
  - public abstract void Setup(ScriptableRenderContext context, ref RenderingData renderingData);

- ScriptableRendererFeature
RenderingData
RenderPassEvent
RenderTargetHandle
```


## SubmitRenderRequest

- https://docs.unity3d.com/ScriptReference/Camera.SubmitRenderRequest.html
- https://docs.unity3d.com/ScriptReference/Rendering.RenderPipeline.SubmitRenderRequest.html
  - UniversalRenderPipeline은 다음을 지원합니다.
    - ScriptableRenderer.StandardRequest: 이 요청 유형은 전체 URP 카메라 스택을 렌더링하고 결과를 지정된 대상에 출력합니다. Base Camera에서만 호출할 수 있습니다.
    - UniversalRenderPipeline.SingleCameraRequest: 이 요청 유형은 단일 URP 카메라를 렌더링하고 그 결과를 지정된 대상에 출력합니다.
- https://docs.unity3d.com/ScriptReference/Rendering.RenderPipeline.ProcessRenderRequests.html

SubmitRenderRequest하면 파이프라인의 Rendering.RenderPipeline.ProcessRenderRequests 이 실행됨.


## Ref

- <https://github.com/cinight/CustomSRP/tree/master/Assets>
- <https://blogs.unity3d.com/2018/01/31/srp-overview/>
- <https://blogs.unity3d.com/kr/2019/02/28/srp-batcher-speed-up-your-rendering/>
- <https://docs.unity3d.com/Manual/ScriptableRenderPipeline.html>
- <https://docs.unity3d.com/Manual/srp-creating-render-pipeline-asset-and-render-pipeline-instance.html>
- <https://github.com/cinight/CustomSRP>
- [catlikecoding - Custom SRP](https://catlikecoding.com/unity/tutorials/custom-srp/)

- [2019 - Unity SRP와 LWRP에 대한 모든 것!](https://www.youtube.com/watch?v=MuzLdCXoJ9I)
- [2020 - Universal RenderPipeline의 Custom RenderPass를 활용하여 렌더링 기능을 구현해보자 Track1-2](https://www.youtube.com/watch?v=vtfe3UgDs0w)
- [2020 - Dev Weeks: URP 기본 구성과 흐름](https://www.youtube.com/watch?v=QRlz4-pAtpY)
  - [Dev Weeks 2020.5. 세션자료](http://www.unitysquare.co.kr/growwith/resource/form?id=83)
- [2020 - Dev Weeks: URP 셰이더 뜯어보기](https://www.youtube.com/watch?v=9K1uOihvNyg)
  - [Dev Weeks 2020.6. 세션자료](http://www.unitysquare.co.kr/growwith/resource/form?id=87)
