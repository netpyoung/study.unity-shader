

resourceData.activeColorTexture 를 1/4배까지 다운샘플링하고 다시 업샘플링하여 원복시킴.

``` cs
private class PassData
{
    internal TextureHandle source;             // resourceData.activeColorTexture
    internal TextureHandle destination;        // 1
    internal TextureHandle destinationHalf;    // 1/2
    internal TextureHandle destinationQuarter; // 1/4
}

public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    string passName = "Unsafe Pass";

    using (IUnsafeRenderGraphBuilder builder = renderGraph.AddUnsafePass(passName, out PassData passData))
    {
        UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

        TextureDesc descriptor;
        {
            descriptor = passData.source.GetDescriptor(renderGraph);
            descriptor.msaaSamples = MSAASamples.None;
            descriptor.clearBuffer = false;
        }

        TextureHandle destination;
        {
            descriptor.name = "UnsafeTexture";
            destination = renderGraph.CreateTexture(descriptor);
        }


        TextureHandle destinationHalf;
        {
            descriptor.width /= 2;
            descriptor.height /= 2;
            descriptor.name = "UnsafeTexture2";
            destinationHalf = renderGraph.CreateTexture(descriptor);
        }

        TextureHandle destinationQuarter;
        {
            descriptor.width /= 2;
            descriptor.height /= 2;
            descriptor.name = "UnsafeTexture3";
            destinationQuarter = renderGraph.CreateTexture(descriptor);
        }

        passData.source = resourceData.activeColorTexture;
        passData.destination = destination;
        passData.destinationHalf = destinationHalf;
        passData.destinationQuarter = destinationQuarter;

        builder.UseTexture(passData.source);
        builder.UseTexture(passData.destination, AccessFlags.WriteAll);
        builder.UseTexture(passData.destinationHalf, AccessFlags.WriteAll);
        builder.UseTexture(passData.destinationQuarter, AccessFlags.WriteAll);

        builder.AllowPassCulling(value: false);

        builder.SetRenderFunc((PassData data, UnsafeGraphContext context) => ExecutePass(data, context));
    }
}

static void ExecutePass(PassData data, UnsafeGraphContext context)
{
    CommandBuffer unsafeCmd = CommandBufferHelpers.GetNativeCommandBuffer(context.cmd);

    // 1 => 1
    context.cmd.SetRenderTarget(data.destination);
    Blitter.BlitTexture(unsafeCmd, data.source, new Vector4(1, 1, 0, 0), 0, false);

    // 1 => 1/2
    context.cmd.SetRenderTarget(data.destinationHalf);
    Blitter.BlitTexture(unsafeCmd, data.destination, new Vector4(1, 1, 0, 0), 0, false);

    // 1/2 => 1/4
    context.cmd.SetRenderTarget(data.destinationQuarter);
    Blitter.BlitTexture(unsafeCmd, data.destinationHalf, new Vector4(1, 1, 0, 0), 0, false);

    // 1/4 => 1/2
    context.cmd.SetRenderTarget(data.destinationHalf);
    Blitter.BlitTexture(unsafeCmd, data.destinationQuarter, new Vector4(1, 1, 0, 0), 0, false);

    // 1/2 => 1
    context.cmd.SetRenderTarget(data.destination);
    Blitter.BlitTexture(unsafeCmd, data.destinationHalf, new Vector4(1, 1, 0, 0), 0, false);
}
```

- unsafe에서는 UseTexture를 써야하고 다음은 안됨
  - builder.UseTextureFragment
  - builder.UseTextureFragmentDepth