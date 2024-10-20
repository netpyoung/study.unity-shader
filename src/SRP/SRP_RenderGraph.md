
# RenderGraph


- [URP 17へのアップグレードとRender Graphの活用方法](https://www.youtube.com/watch?v=jATxYUVIlH0)
  - https://github.com/robin-boucher/URPRendererFeatureExample6
- RasterRenderPass
  - 1개 렌더링 타겟
 

`Edit > Project Settings > Graphics Piepline Specific Settings > Render Graph > Compatibility Mode (Render Graph Disabled)`

일단 캡쳐된걸 가져온다

일단 꺼두자
```cs
ScriptableRenderPass
  public bool requiresIntermediateTexture { get; set; }
// 활성화되면 렌더링 과정에서 렌더 타겟을 직접 출력 프레임 버퍼로 쓰지 않고, 중간 텍스처에 렌더링한 뒤 후처리 또는 복사 후 최종 프레임 버퍼로 전달합니다.
// 후처리(Post-processing)와 필터 적용 지원
// 해상도 차이 해결
// Setting this property to true forces rendering of all passes in the URP frame via an intermediate texture.
// Use this option for passes that do not support rendering directly to the backbuffer or that require sampling the active color target.
// Using this option might have a significant performance impact on untethered VR platforms.
  
  
  this.renderTargetId = Shader.PropertyToID(this.renderTargetName);

```

활성 스크린 칼러 얻고

``` cs
// resourceData
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
  UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();
  if (resourceData.isActiveTargetBackBuffer)
  {
      return;
  }

  TextureHandle srcCamColor = resourceData.activeColorTexture;
  if (!srcCamColor.IsValid())
  {
      return;
  }
```

IRasterRenderGraphBuilder를 생성

``` cs
// 래스터화는 3D 모델의 정점(Vertices) 데이터를 2D 픽셀 데이터로 변환하는 과정입니다.
using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(passName, out PassData passData))
{

  TextureHandle cameraColorTextureHandle = resourceData.activeColorTexture;
  builder.UseTexture(cameraColorTextureHandle, AccessFlags.Read);

  RenderTextureDescriptor desc = cameraData.cameraTargetDescriptor;
  desc.msaaSamples = 1;
  desc.depthBufferBits = 0;
  TextureHandle targetTextureHandle = UniversalRenderer.CreateRenderGraphTexture(renderGraph, desc, this.renderTargetName, true);
  builder.SetRenderAttachment(targetTextureHandle, AccessFlags.Write);
  
  builder.SetGlobalTextureAfterPass(targetTextureHandle, this.renderTargetId);

  builder.SetRenderFunc(ExecutePass);

```


``` cs
 private static void ExecutePass(PassData passData, RasterGraphContext graphContext)
{
  Blitter.BlitTexture(cmd, passData.sourceTextureHandle, new Vector4(1, 1, 0, 0), passData.material, 0);
```
                    

## 잘 모르겠는거

RenderingUtils.ReAllocateHandleIfNeeded(ref renderTextureHandle
RTHandle renderTextureHandle = RTHandles.Alloc(texture); // 외부 텍스처에서 렌더 텍스처 핸들을 생성합니다.

TextureHandle texture = renderGraph.ImportTexture(renderTextureHandle); // 텍스처를 가져와서 객체를 렌더 그래프 시스템에서 사용할 수 있는 객체 RTHandle로 변환합니다

Dispose => renderTextureHandle.Release();




TextureDesc destinationDesc = renderGraph.GetTextureDesc(source);
TextureHandle destination = renderGraph.CreateTexture(destinationDesc);  
renderGraph.AddCopyPass(resourceData.activeColorTexture, destination, passName: passName);




``` cs
// 래스터화는 3D 모델의 정점(Vertices) 데이터를 2D 픽셀 데이터로 변환하는 과정입니다.
using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(passName, out PassData passData))
{
  passData.copySourceTexture = resourceData.activeColorTexture;
  UniversalCameraData cameraData = frameData.Get<UniversalCameraData>();
  RenderTextureDescriptor desc = cameraData.cameraTargetDescriptor;
  desc.msaaSamples = 1; // Set msaaSamples to 1 to get a non-multisampled destination texture.
  desc.depthBufferBits = 0; // Set depthBufferBits to 0 to ensure that the CreateRenderGraphTexture method creates a color texture and not a depth texture.

  // 제 GPU 리소스 할당을 의미하기보다는 렌더 그래프에 사용할 리소스의 의도를 명시하는 역할을 합니다.
  TextureHandle destination = UniversalRenderer.CreateRenderGraphTexture(renderGraph, desc, "CopyTexture", false);
  builder.UseTexture(passData.copySourceTexture); // Declare that this pass uses the input texture.
  builder.SetRenderAttachment(destination, 0); //         /// Use the texture as an rendertarget attachment.


  builder.AllowPassCulling(false);
  builder.SetRenderFunc<PassData>(ExecutePass);
}
```



``` csharp
ScriptableRendererFeature
// Create:
//   - 렌더러 기능이 처음 로드될 때.
//   - 렌더러 기능을 활성화하거나 비활성화하는 경우.
//   - 렌더러 기능의 검사기에서 속성을 변경할 때.

// AddRenderPasses:
//   - 매 프레임마다, 각 카메라마다 한 번씩 호출합니다.




// 이전 렌더그래프 API
ScriptableRenderPass
    [Obsolete(DeprecationMessage.CompatibilityScriptingAPIObsolete, false)]
    Execute	Use
    OnCameraSetup
    OnCameraCleanup



// 새로운 렌더그래프 API
ScriptableRenderPass
// RecordRenderGraph
//  - 이 메서드는 렌더 그래프에서 렌더 패스를 추가하고 구성합니다.
//  - 이 프로세스에는 렌더 패스 입력 및 출력을 선언하는 것이 포함되지만 명령 버퍼에 명령을 추가하는 것은 포함되지 않습니다.
//  - Unity는 각 카메라에 대해 한 번씩 매 프레임마다 이 메서드를 호출합니다.
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
```

https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@17.0/manual/render-graph-system.html

## https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@17.0/manual/renderer-features/create-custom-renderer-feature.html

- ScriptableRendererFeature
  - ScriptableRenderPass 를 매 프레임마다 큐에 넣음
- ScriptableRenderPass
  - RenderTextureDescriptor API를 이용하여 임시 렌더 텍스쳐를 만듬
  - TextureHandle and the Blitter API 를 이용하여 [셰이더 적용](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@17.0/manual/renderer-features/create-custom-renderer-feature.html#example-shader)


## Render Requests

https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@17.0/manual/User-Render-Requests.html

## 


class
    RenderGraph
    ContextContainer
    ContextItem

TODO

| 렌더 그래프가 관리하는 리소스 유형 |        |                                            |                                                                                                                                   |
| ---------------------------------- | ------ | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| RTHandles                          | class  | RenderTexture API 감싼것                   | <https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@12.0/api/UnityEngine.Rendering.RTHandles.html>                 |
| ComputeBuffers                     | class  | Compute Shader를 위한 GPU data buffer      | <https://docs.unity3d.com/ScriptReference/ComputeBuffer.html>                                                                     |
| RendererLists                      | struct | 렌더링에 사용하는 셋팅같은 정보들 모아둔것 | <https://docs.unity3d.com/Packages/com.unity.render-pipelines.core@12.0/api/UnityEngine.Experimental.Rendering.RendererList.html> |

- <https://github.com/cinight/CustomSRP/tree/master/Assets/SRP0802_RenderGraph>






## RenderGraph - cont

- https://github.com/alexuhui/MySRP/blob/main/Assets/Custom%20RP/ShaderLibrary/Common.hlsl
- https://catlikecoding.com/unity/custom-srp/2-0-0/

``` cs
using UnityEngine.Experimental.Rendering.RenderGraphModule;

readonly RenderGraph renderGraph = new("Custom SRP Render Graph");

protected override void Dispose(bool disposing)
{
    …
    renderGraph.Cleanup();
}

var renderGraphParameters = new RenderGraphParameters
{
    commandBuffer = CommandBufferPool.Get(),
    currentFrameIndex = Time.frameCount,
    executionName = "Render Camera",
    scriptableRenderContext = context
};

using (renderGraph.RecordAndExecute(renderGraphParameters))
{

}
CommandBufferPool.Release(renderGraphParameters.commandBuffer);
renderGraph.EndFrame();

```




         private static void ExecutePass(PassData passData, UnsafeGraphContext graphContext)
            {
                CommandBuffer cmd = CommandBufferHelpers.GetNativeCommandBuffer(graphContext.cmd);
