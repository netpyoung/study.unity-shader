
활성 color 텍스쳐를 새로운 텍스쳐로 복사하는 예

``` cs
public class CopyRenderFeature : ScriptableRendererFeature
{
    CopyRenderPass m_CopyRenderPass;

    public override void Create()
    {
        m_CopyRenderPass = new CopyRenderPass();
        m_CopyRenderPass.renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
    }

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
    {
        renderer.EnqueuePass(m_CopyRenderPass);
    }
}
```

``` cs
class CopyRenderPass : ScriptableRenderPass
{
    public CopyRenderPass()
    {
        requiresIntermediateTexture = true;
    }

    public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
    {
        const string passName = "Copy To or From Temp Texture";

        UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

        TextureHandle srcHandle = resourceData.activeColorTexture;
        TextureHandle dstHandle;
        {
            TextureDesc destinationDesc = renderGraph.GetTextureDesc(srcHandle);
            destinationDesc.name = $"CameraColor-{passName}";
            destinationDesc.clearBuffer = false;
            dstHandle = renderGraph.CreateTexture(destinationDesc);
        }

        if (RenderGraphUtils.CanAddCopyPassMSAA())
        {
            renderGraph.AddCopyPass(srcHandle, dstHandle, passName: passName);
            renderGraph.AddCopyPass(dstHandle, srcHandle, passName: passName);
        }
        else
        {
            Debug.Log("Can't add the copy pass due to MSAA");
        }
    }
}
```

### requiresIntermediateTexture

| requiresIntermediateTexture |                                                                                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| true                        | 메라의 렌더링 결과를 최종 프레임 버퍼로 직접 렌더링하지 않고, 먼저 Intermediate Texture(임시 텍스처)에 렌더링한 후 최종 출력으로 전달. |
| false                       | 바로 최종 프레임 버퍼로 렌더링을 수행.                                                                                                 |

- RenderFeature가 아니라 ScriptableRenderPass에서 셋팅하는게 good practice
- 


### UniversalResourceData

- 텍스쳐 핸들을 포함하고 있음.
  - active color / depth texture 등등.
- isActiveTargetBackBuffer
  - pass의 requiresIntermediateTexture가 true면 resourceData의 isActiveTargetBackBuffer는 무조껀 false



### renderGraph

renderGraph.AddCopyPass(srcHandle, dstHandle, passName: passName);

### RenderGraphUtils

RenderGraphUtils.CanAddCopyPassMSAA()