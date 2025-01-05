```cs
[Flags]
public enum ScriptableRenderPassInput
{
    None   = 0,
    Depth  = 1 << 0,
    Normal = 1 << 1,
    Color  = 1 << 2,
    Motion = 1 << 3,
}
```

EnqueuePass로 pass를 넣기전 pass의 ConfigureInput(ScriptableRenderPassInput passInput)를 호출

``` cs
// TextureHandle
//     resourceData.cameraOpaqueTexture
//     resourceData.cameraDepthTexture
//     resourceData.cameraNormalsTexture
//     resourceData.motionVectorColor
//     TextureHandle.nullHandle

public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

    TextureHandle source = GetTextureHandleFromType(resourceData, m_TextureType);
    if (!source.IsValid())
    {
        Debug.Log("Input texture is not created. Likely the pass event is before the creation of the resource. Skipping OutputTexturePass.");
        return;
    }

    RenderGraphUtils.BlitMaterialParameters para = new RenderGraphUtils.BlitMaterialParameters(source, resourceData.activeColorTexture, m_Material, shaderPass: 0);
    param.sourceTexturePropertyID = Shader.PropertyToID(m_TextureName);
    renderGraph.AddBlitPass(param, passName: "Blit Selected Resource");
}
```