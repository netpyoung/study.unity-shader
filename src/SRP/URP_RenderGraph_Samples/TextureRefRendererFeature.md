[BlitRendererFeature](BlitRendererFeature.md)


ContextContainer frameData 동작방식

- 2 ScriptableRenderPass
  - UpdateRefPass
  - CopyBackRefPass
- ContextItem를 상속받는 TexRefData를 이용
  - UpdateRefPass에서 resourceData.activeColorTexture => texRef.texture
    - UpdateRefPass에서 foreach (Material mat in m_DisplayMaterials) 하므로 매 프레임 첫 루프 단계에서는 frameData.Contains<TexRefData>()가 false, 그 후로는 true.
  - CopyBackRefPass에서 texRef.texture => resourceData.activeColorTexture


``` cs
public class TexRefData : ContextItem
{
    public TextureHandle texture = TextureHandle.nullHandle;

    public override void Reset()
    {
        texture = TextureHandle.nullHandle;
    }
}
```

``` cs
//     class UpdateRefPass : ScriptableRenderPass

public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    foreach (Material mat in m_DisplayMaterials)
    {
        if (mat == null)
        {
            Debug.LogWarning($"Skipping render pass for unassigned material.");
            continue;
        }

        using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass($"UpdateRefPass_{mat.name}", out PassData passData))
        {

            TexRefData texRef;
            if (frameData.Contains<TexRefData>())
            {
                texRef = frameData.Get<TexRefData>();
            }
            else
            {
                UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();
                texRef = frameData.Create<TexRefData>();
                texRef.texture = resourceData.activeColorTexture;
            }

            TextureDesc descriptor = texRef.texture.GetDescriptor(renderGraph);
            descriptor.msaaSamples = MSAASamples.None;
            descriptor.name = $"BlitMaterialRefTex_{mat.name}";
            descriptor.clearBuffer = false;

            TextureHandle srcHandle = texRef.texture;
            TextureHandle dstHandle = renderGraph.CreateTexture(descriptor);
            texRef.texture = dstHandle;

            passData.source = srcHandle;
            passData.destination = dstHandle;
            passData.material = mat;


            builder.UseTexture(input: passData.source);
            builder.SetRenderAttachment(tex: passData.destination, index: 0);
            builder.SetRenderFunc((PassData data, RasterGraphContext rgContext) => ExecutePass(data, rgContext));
        }
    }
}

static void ExecutePass(PassData data, RasterGraphContext rgContext)
{
    Blitter.BlitTexture(rgContext.cmd, data.source, scaleBias, data.material, pass: 0);
}
```

``` cs
// class CopyBackRefPass : ScriptableRenderPass

public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    if (!frameData.Contains<TexRefData>())
    {
        return;
    }

    TexRefData texRef = frameData.Get<TexRefData>();
    TextureHandle srcHandle = texRef.texture;

    UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();
    TextureHandle dstHandle = resourceData.activeColorTexture;

    renderGraph.AddBlitPass(srcHandle, dstHandle, scale: Vector2.one, offset: Vector2.zero, passName: "Blit Back Pass");
}
```

### ContextContainer frameData

``` cs


bool isExist_TexRefData = frameData.Contains<TexRefData>();
texRef = frameData.GetOrCreate<TexRefData>();
texRef = frameData.Create<TexRefData>();
texRef = frameData.Get<TexRefData>();


```

### scaleBias

Vector4 scaleBias(scaleX, scaleY, offsetX, offsetY);
Blitter.BlitTexture(rgContext.cmd, data.source, scaleBias, data.material, pass: 0);

| scaleX        |                                   |
| ------------- | --------------------------------- |
| 1.0 < x       | 텍스처가 가로 방향으로 늘어남.    |
| 0.0 < x < 1.0 | 텍스처가 가로 방향으로 줄어듬.    |
| x < 0.0       | 음수 값은 가로 방향을 반전(flip). |

| offsetX |                           |
| ------- | ------------------------- |
| x > 0   | 텍스처가 오른쪽으로 이동. |
| x < 0   | 텍스처가 왼쪽으로 이동.   |


| offsetX |                       |
| ------- | --------------------- |
| x > 0   | 텍스처가 위로 이동.   |
| x < 0   | 텍스처가 아래로 이동. |
