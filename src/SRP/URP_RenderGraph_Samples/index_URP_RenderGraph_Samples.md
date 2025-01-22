
|                                                                               |                                                    |                                                                                                                 |
| ----------------------------------------------------------------------------- | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| [CopyRenderFeature](CopyRenderFeature.md)                                     | renderGraph.AddCopyPass                            | Samples\Universal RP\17.0.3\URP RenderGraph Samples\Blit\CopyRenderFeature.cs                                   |
| [BlitAndSwapColorRendererFeature](BlitAndSwapColorRendererFeature.md)         | renderGraph.AddBlitPass / resourceData.cameraColor | Samples\Universal RP\17.0.3\URP RenderGraph Samples\BlitWithMaterial\BlitAndSwapColorRendererFeature.cs         |
| [BlitRendererFeature](BlitRendererFeature.md)                                 | ContextContainer frameData / ContextItem           | Samples\Universal RP\17.0.3\URP RenderGraph Samples\Blit w. FrameData\BlitRendererFeature.cs                    |
| [TextureRefRendererFeature](TextureRefRendererFeature.md)                     | ContextContainer frameData / ContextItem           | Samples\Universal RP\17.0.3\URP RenderGraph Samples\TextureReference w. FrameData\TextureRefRendererFeature.cs  |
| [OutputTextureRendererFeature](OutputTextureRendererFeature.md)               | ConfigureInput                                     | Samples\Universal RP\17.0.3\URP RenderGraph Samples\OutputTexture\OutputTextureRendererFeature.cs               |
| .                                                                             |                                                    |                                                                                                                 |
| [UnsafePassRenderFeature](UnsafePassRenderFeature.md)                         |                                                    | Samples\Universal RP\17.0.3\URP RenderGraph Samples\UnsafePass\UnsafePassRenderFeature.cs                       |
| .                                                                             |                                                    |                                                                                                                 |
| [FrameBufferFetchRenderFeature](FrameBufferFetchRenderFeature.md)             |                                                    | Samples\Universal RP\17.0.3\URP RenderGraph Samples\FramebufferFetch\FrameBufferFetchRenderFeature.cs           |
| [RendererListRenderFeature](RendererListRenderFeature.md)                     |                                                    | Samples\Universal RP\17.0.3\URP RenderGraph Samples\RendererList\RendererListRenderFeature.cs                   |
| .                                                                             |                                                    |                                                                                                                 |
| [GbufferVisualizationRendererFeature](GbufferVisualizationRendererFeature.md) |                                                    | Samples\Universal RP\17.0.3\URP RenderGraph Samples\GbufferVisualization\GbufferVisualizationRendererFeature.cs |
| [GlobalGbuffersRendererFeature](GlobalGbuffersRendererFeature.md)             |                                                    | Samples\Universal RP\17.0.3\URP RenderGraph Samples\GlobalGbuffers\GlobalGbuffersRendererFeature.cs             |
| .                                                                             |                                                    |                                                                                                                 |
| [ComputeRendererFeature](ComputeRendererFeature.md)                           |                                                    | Samples\Universal RP\17.0.3\URP RenderGraph Samples\Compute\ComputeRendererFeature.cs                           |
| [MrtRendererFeature](MrtRendererFeature.md)                                   |                                                    | Samples\Universal RP\17.0.3\URP RenderGraph Samples\MRT\MrtRendererFeature.cs                                   |


## quick sheet

| IBaseRenderGraphBuilder                                                                               |                                    |
| ----------------------------------------------------------------------------------------------------- | ---------------------------------- |
| buidler.UseTexture(input: passData.source, flag: AccessFlags.Read)                                    | 입력                               |
| buidler.SetRenderAttachment(tex: passData.destination, index: 0)                                      |                                    |
| buidler.SetRenderFunc<PassData>(ExecutePass)                                                          | 렌더함수 지정                      |
| buidler.UseRendererList(input: rendererListHandle)                                                    | RendererListHandle는 항상 readonly |
| buidler.AllowPassCulling(value: false)                                                                |                                    |
| buidler.SetGlobalTextureAfterPass(input: textureHandle, propertyId: Shader.PropertyToID("_HelloTex")) |                                    |





context.cmd.SetRenderTarget(data.TmpCopyTexHandle);                       // Output이라 보면 됨
Blitter.BlitTexture(nativeCmd, textureHandle, scaleBias, mipLevel: 0, bilinear: false);
Blitter.BlitTexture(nativeCmd, textureHandle, scaleBias, material, pass); // => Blitter.BlitTexture함수를 쓰면 inputTextureHandle이 shader에서 _BlitTexture로 바인딩됨.


RenderTextureDescriptor rtdesc2 = new RenderTextureDescriptor(w, h, GraphicsFormat.R16G16_SFloat, 0)
{
    autoGenerateMips = true,
    useMipMap = true
};
RenderingUtils.ReAllocateHandleIfNeeded(ref _tmpCurrMipmapRT, rtdesc2, FilterMode.Bilinear, TextureWrapMode.Clamp, name: "_tmpCurrMipmapRT");



CommandBuffer nativeCmd = CommandBufferHelpers.GetNativeCommandBuffer(context.cmd);






### migration RenderGraph API

- RenderingUtils.ReAllocateIfNeeded => RenderingUtils.ReAllocateHandleIfNeeded

- RTHandle 보다는 TextureHandle

``` cs
textureHandle = renderGraph.ImportTexture(rtHandle);
```

- ScriptableRendererFeature를 상속받아쓰는 SetupRenderPasses 더 이상 안쓰임.


``` cs
// before
public override void Configure(CommandBuffer cmd, RenderTextureDescriptor cameraTextureDescriptor)
{
    ConfigureInput(ScriptableRenderPassInput.Normal);
}

// after
using (IUnsafeRenderGraphBuilder builder = renderGraph.AddUnsafePass(passName, out PassData passData))
{
    ConfigureInput(ScriptableRenderPassInput.Normal);
}
```