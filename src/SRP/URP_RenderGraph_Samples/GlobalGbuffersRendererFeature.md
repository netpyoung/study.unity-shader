


셰이더에서 텍스쳐 사용시 builder.UseTexture를 사용해야 하는데, builder.SetGlobalTextureAfterPass로 미리 등록시키면 ` builder.UseAllGlobalTextures(enable: true) `로 등록된 텍스쳐들을 사용할 수 있다.

|           |                               |
| --------- | ----------------------------- |
| _GBuffer2 | _CameraNormalsTexture         |
| _GBuffer4 | _CameraDepthTexture           |
| _GBuffer5 | _CameraRenderingLayersTexture |

``` cs
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    UniversalRenderingData universalRenderingData = frameData.Get<UniversalRenderingData>();
    if (universalRenderingData.renderingMode != RenderingMode.Deferred)
    {
        return;
    }

    UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();
    TextureHandle[] gBuffer = resourceData.gBuffer;

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(m_PassName, out PassData passData))
    {
        builder.AllowPassCulling(value: false);
        SetGlobalGBufferTextures(builder, gBuffer);
        builder.SetRenderFunc((PassData data, RasterGraphContext context) => { /* nothing to be rendered */ });
    }
}

private void SetGlobalGBufferTextures(IRasterRenderGraphBuilder builder, TextureHandle[] gBuffer)
{
    for (int i = 0; i < gBuffer.Length; i++)
    {
        if (i != GbufferLightingIndex && gBuffer[i].IsValid())
        {
            builder.SetGlobalTextureAfterPass(gBuffer[i], s_GBufferShaderPropertyIDs[i]);
        }
    }

    builder.UseAllGlobalTextures(enable: true);

    if (gBuffer[GBufferNormalSmoothnessIndex].IsValid())
    {
        builder.SetGlobalTextureAfterPass(gBuffer[GBufferNormalSmoothnessIndex], Shader.PropertyToID("_CameraNormalsTexture"));
    }

    if (gBuffer[GbufferDepthIndex].IsValid())
    {
        builder.SetGlobalTextureAfterPass(gBuffer[GbufferDepthIndex], Shader.PropertyToID("_CameraDepthTexture"));
    }

    if (GBufferRenderingLayersIndex < gBuffer.Length && gBuffer[GBufferRenderingLayersIndex].IsValid())
    {
        builder.SetGlobalTextureAfterPass(gBuffer[GBufferRenderingLayersIndex], Shader.PropertyToID("_CameraRenderingLayersTexture"));
    }
}
```

### IRasterRenderGraphBuilder builder

``` cs
builder.AllowPassCulling(value: false);

builder.SetGlobalTextureAfterPass(in TextureHandle input, int propertyId);
```