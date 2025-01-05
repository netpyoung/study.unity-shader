renderGraph.AddBlitPass를 이용 후 resourceData.cameraColor로 셋팅

``` cs
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

    if (resourceData.isActiveTargetBackBuffer)
    {
        Debug.LogError($"Skipping render pass. BlitAndSwapColorRendererFeature requires an intermediate ColorTexture, we can't use the BackBuffer as a texture input.");
        return;
    }

    TextureHandle srcHandle = resourceData.activeColorTexture;
    TextureHandle dstHandle;
    {
        TextureDesc destinationDesc = renderGraph.GetTextureDesc(srcHandle);
        destinationDesc.name = $"CameraColor-{m_PassName}";
        destinationDesc.clearBuffer = false;
        dstHandle = renderGraph.CreateTexture(destinationDesc);
    }


    RenderGraphUtils.BlitMaterialParameters blitParam = new RenderGraphUtils.BlitMaterialParameters(srcHandle, dstHandle, m_BlitMaterial, shaderPass: 0);
    renderGraph.AddBlitPass(blitParam, passName: m_PassName);


    resourceData.cameraColor = dstHandle;
}
```


## renderGraph

RenderGraphUtils.BlitMaterialParameters blitParam = new RenderGraphUtils.BlitMaterialParameters(srcHandle, dstHandle, m_BlitMaterial, shaderPass: 0);
renderGraph.AddBlitPass(blitParam, passName: m_PassName);
