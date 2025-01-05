
- BlitAndSwapColorRendererFeature.cs 예제와의 차이는
  - requiresIntermediateTexture 를 키지 않았고
  - BlitData를 이용.
  - pass를 3단계
    1. BlitStartRenderPass : 버퍼만들고 activeColorTexture를 m_Texture에 저장
    2. BlitRenderPass : 버퍼 사용해서 m_TextureHandleFront / m_TextureHandleBack 을 blit.
    3. BlitEndRenderPass : 버퍼 사용해서 m_Texture를 activeColorTexture에 저장

``` cs
class BlitStartRenderPass : ScriptableRenderPass
{
    public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
    {
        BlitData blitTextureData = frameData.Create<BlitData>();
        blitTextureData.RecordBlitColor(renderGraph, frameData);
    }
}

class BlitEndRenderPass : ScriptableRenderPass
{
    public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
    {
        BlitData blitTextureData = frameData.Get<BlitData>();
        blitTextureData.RecordBlitBackToColor(renderGraph, frameData);
    }
}

class BlitRenderPass : ScriptableRenderPass
{
    public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
    {
        BlitData blitTextureData = frameData.Get<BlitData>();

        foreach (Material material in m_Materials)
        {
            if (material == null)
            {
                continue;
            }

            blitTextureData.RecordFullScreenPass(renderGraph, $"Blit {material.name} Pass", material);
        }
    }
}
```

``` cs
public class BlitData : ContextItem, IDisposable
{
    // ContextItem.Reset()
    // 텍스쳐 핸들이 1프레임에만 살아있기에 invalid texture handles에 의한 릭을 피하기 위해 매 프레임 후에 texture handle을 리셋시켜야함.
    public override void Reset()
    {
        m_TextureHandleFront = TextureHandle.nullHandle;
        m_TextureHandleBack = TextureHandle.nullHandle;
        m_Texture = TextureHandle.nullHandle;
        m_IsFront = true;
    }

    // IDisposable.Dispose()
    public void Dispose()
    {
        if (m_TextureFront != null)
        {
            m_TextureFront.Release();
        }
        
        if (m_TextureFront != null)
        {
            m_TextureFront.Release();
        }
    }


    // BlitStartRenderPass => RecordBlitColor
    // BlitRenderPass      => RecordFullScreenPass
    // BlitEndRenderPass   => RecordBlitBackToColor
}
```

``` cs
// BlitStartRenderPass => RecordBlitColor
public void RecordBlitColor(RenderGraph renderGraph, ContextContainer frameData)
{
    if (!m_Texture.IsValid())
    {
        UniversalCameraData cameraData = frameData.Get<UniversalCameraData>();

        RenderTextureDescriptor descriptor = cameraData.cameraTargetDescriptor;
        descriptor.msaaSamples = 1;
        descriptor.depthStencilFormat = UnityEngine.Experimental.Rendering.GraphicsFormat.None;

        string texName = "_BlitTextureData";

        RenderingUtils.ReAllocateHandleIfNeeded(ref m_TextureFront, descriptor, FilterMode.Bilinear, TextureWrapMode.Clamp, name: $"{texName}_Front");
        RenderingUtils.ReAllocateHandleIfNeeded(ref m_TextureBack, descriptor, FilterMode.Bilinear, TextureWrapMode.Clamp, name: $"{texName}_Back");

        m_TextureHandleFront = renderGraph.ImportTexture(m_TextureFront);
        m_TextureHandleBack = renderGraph.ImportTexture(m_TextureBack);
        m_Texture = m_TextureHandleFront;
    }

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass("BlitColorPass", out PassData passData))
    {
        UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

        passData.source = resourceData.activeColorTexture;
        passData.destination = m_Texture;
        passData.material = null;

        builder.UseTexture(passData.source);
        builder.SetRenderAttachment(passData.destination, index: 0);
        builder.SetRenderFunc((PassData passData, RasterGraphContext rgContext) => ExecutePass(passData, rgContext));
    }
}
```

``` cs
// BlitRenderPass => RecordFullScreenPass
public void RecordFullScreenPass(RenderGraph renderGraph, string passName, Material material)
{
    if (!m_Texture.IsValid())
    {
        Debug.LogWarning("Invalid input texture handle, will skip fullscreen pass.");
        return;
    }

    if (material == null)
    {
        return;
    }

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(passName, out PassData passData))
    {
        m_IsFront = !m_IsFront;

        TextureHandle src = m_Texture;
        TextureHandle dst;
        if (m_IsFront)
        {
            dst = m_TextureHandleFront;
        }
        else
        {
            dst = m_TextureHandleBack;
        }

        m_Texture = dst;

        passData.source = src;
        passData.destination = dst;
        passData.material = material;

        builder.UseTexture(passData.source);
        builder.SetRenderAttachment(passData.destination, index: 0);
        builder.SetRenderFunc((PassData passData, RasterGraphContext rgContext) => ExecutePass(passData, rgContext));
    }
}
```

``` cs
// BlitEndRenderPass => RecordBlitBackToColor

public void RecordBlitBackToColor(RenderGraph renderGraph, ContextContainer frameData)
{
    if (!m_Texture.IsValid())
    {
        return;
    }

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass($"BlitBackToColorPass", out PassData passData))
    {
        UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

        passData.source = m_Texture;
        passData.destination = resourceData.activeColorTexture;
        passData.material = null;

        builder.UseTexture(passData.source);
        builder.SetRenderAttachment(passData.destination, index: 0);
        builder.SetRenderFunc((PassData passData, RasterGraphContext rgContext) => ExecutePass(passData, rgContext));
    }
}

```

``` cs
static void ExecutePass(PassData data, RasterGraphContext rgContext)
{
    if (data.material == null)
    {
        Blitter.BlitTexture(rgContext.cmd, data.source, scaleBias, mipLevel: 0, bilinear: false);
        return;
    }

    Blitter.BlitTexture(rgContext.cmd, data.source, scaleBias, data.material, pass: 0);
}
```

### ContextContainer frameData

``` cs
BlitData blitTextureData = frameData.Create<BlitData>();



BlitData blitTextureData = frameData.Get<BlitData>();
```

### IRasterRenderGraphBuilder builder

``` cs
builder.UseTexture(passData.source);                     // set input
builder.SetRenderAttachment(passData.destination, 0);    // set output
builder.SetRenderFunc((PassData passData, RasterGraphContext rgContext) => ExecutePass(passData, rgContext));



using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(passName, out PassData passData))
using (IComputeRenderGraphBuilder builder = renderGraph.AddComputePass(passName, out PassData passData))
using (IUnsafeRenderGraphBuilder builder = renderGraph.AddUnsafePass(passName, out PassData passData))
```

### Etc

``` cs
// https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/Runtime/Textures/MSAASamples.cs
public enum MSAASamples
{
    /// <summary>No MSAA.</summary>
    None = 1,
    /// <summary>MSAA 2X.</summary>
    MSAA2x = 2,
    /// <summary>MSAA 4X.</summary>
    MSAA4x = 4,
    /// <summary>MSAA 8X.</summary>
    MSAA8x = 8
}
```
