
지정된 LayerMask에 있는 오브젝트만 그리도록 RenderList 사용.

renderGraph.CreateRendererList()로 RendererListHandle을 만듬.

람다대신 그냥 넣는게 나을듯?

builder.SetRenderFunc((PassData data, RasterGraphContext context) => ExecutePass(data, context));
builder.SetRenderFunc<PassData>(ExecutePass);


``` cs

public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    string passName = "RenderList Render Pass";

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(passName, out PassData passData))
    {
        UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

        InitRendererLists(renderGraph, frameData, ref passData);

        if (!passData.rendererListHandle.IsValid())
        {
            return;
        }

        builder.UseRendererList(passData.rendererListHandle);
        builder.SetRenderAttachment(resourceData.activeColorTexture, index: 0);
        builder.SetRenderAttachmentDepth(resourceData.activeDepthTexture, AccessFlags.Write);
        builder.SetRenderFunc((PassData data, RasterGraphContext context) => ExecutePass(data, context));
    }
}


static void ExecutePass(PassData passData, RasterGraphContext context)
{
    context.cmd.ClearRenderTarget(clearFlags: RTClearFlags.Color, backgroundColor: Color.green, depth: 1, stencil: 0);
    context.cmd.DrawRendererList(passData.rendererListHandle);
}
```

``` cs
private class PassData
{
    public RendererListHandle rendererListHandle;
}

private void InitRendererLists(RenderGraph renderGraph, ContextContainer frameData, ref PassData passData)
{
    UniversalRenderingData universalRenderingData = frameData.Get<UniversalRenderingData>();

    RendererListParams param;
    {
        DrawingSettings drawSettings;
        {
            m_ShaderTagIdList.Clear();
            ShaderTagId[] forwardOnlyShaderTagIds = new ShaderTagId[]
                {
                new ShaderTagId("UniversalForwardOnly"),
                new ShaderTagId("UniversalForward"),
                new ShaderTagId("SRPDefaultUnlit"), // Legacy shaders (do not have a gbuffer pass) are considered forward-only for backward compatibility
                new ShaderTagId("LightweightForward") // Legacy shaders (do not have a gbuffer pass) are considered forward-only for backward compatibility
                };
            m_ShaderTagIdList.AddRange(forwardOnlyShaderTagIds);
            UniversalCameraData cameraData = frameData.Get<UniversalCameraData>();
            UniversalLightData lightData = frameData.Get<UniversalLightData>();
            SortingCriteria sortFlags = cameraData.defaultOpaqueSortFlags;

            drawSettings = RenderingUtils.CreateDrawingSettings(m_ShaderTagIdList, universalRenderingData, cameraData, lightData, sortFlags);
        }
        RenderQueueRange renderQueueRange = RenderQueueRange.opaque;
        FilteringSettings filterSettings = new FilteringSettings(renderQueueRange, m_LayerMask);

        param = new RendererListParams(universalRenderingData.cullResults, drawSettings, filterSettings);
    }
    passData.rendererListHandle = renderGraph.CreateRendererList(param);
}

```