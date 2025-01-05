
- RendererAsset에서 Rendering > Deferred로 변경
- RenderPassEvent.AfterRenderingDeferredLights
- activeColorTexture == _GBuffer3 (Lighting)

``` cs
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    if (m_Material == null)
    {
        return;
    }

    UniversalRenderingData universalRenderingData = frameData.Get<UniversalRenderingData>();
    if (universalRenderingData.renderingMode != RenderingMode.Deferred)
    {
        return;
    }

    UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();
    TextureHandle[] gBuffer = resourceData.gBuffer;

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(m_PassName, out PassData passData))
    {
        passData.material = m_Material;
        passData.gBuffer = gBuffer;

        builder.SetRenderAttachment(resourceData.activeColorTexture, index: 0, flags: AccessFlags.Write); // resourceData.activeColorTexture == _GBuffer3 (Lighting)
        for (int i = 0; i < resourceData.gBuffer.Length; i++)
        {
            if (i == GbufferLightingIndex)
            {
                continue;
            }

            builder.UseTexture(resourceData.gBuffer[i]);
        }

        builder.SetRenderFunc((PassData data, RasterGraphContext context) => ExecutePass(data, context));
    }
}

static void ExecutePass(PassData data, RasterGraphContext context)
{
    for (int i = 0; i < data.gBuffer.Length; i++)
    {
        data.material.SetTexture(s_GBufferShaderPropertyIDs[i], data.gBuffer[i]);
    }
    context.cmd.DrawProcedural(Matrix4x4.identity, data.material, shaderPass: 0, MeshTopology.Triangles, vertexCount: 3, instanceCount: 1);
}
```


### UniversalRenderingData

``` cs
UniversalRenderingData universalRenderingData = frameData.Get<UniversalRenderingData>();
universalRenderingData.renderingMode

public enum RenderingMode
{
    Forward = 0,
    /// <summary>Render all objects and lighting in one pass using a clustered data structure to access lighting data.</summary>
    [InspectorName("Forward+")]
    ForwardPlus = 2,
    Deferred = 1
};
```
### shader

``` hlsl
#include "Packages/com.unity.render-pipelines.core/ShaderLibrary/GlobalSamplers.hlsl"

TEXTURE2D_X(_GBuffer2);

struct Attributes
{
    uint vertexID : SV_VertexID;
    UNITY_VERTEX_INPUT_INSTANCE_ID
};

struct Varyings
{
    float4 positionCS : SV_POSITION;
    float2 texcoord   : TEXCOORD0;

    UNITY_VERTEX_INPUT_INSTANCE_ID
    UNITY_VERTEX_OUTPUT_STEREO
};

Varyings GBufferVisPassVertex(Attributes input)
{
    Varyings output;
    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_TRANSFER_INSTANCE_ID(input, output);
    UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(output);

    float4 pos = GetFullScreenTriangleVertexPosition(input.vertexID);
    float2 uv  = GetFullScreenTriangleTexCoord(input.vertexID);

    output.positionCS = pos;
    output.texcoord   = uv;

    return output;
}

void GBufferVisPassFragment(Varyings input, out half4 outColor : SV_Target0)
{
    UNITY_SETUP_INSTANCE_ID(input);
    UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input);

    float2 uv = input.texcoord;
    #ifndef UNITY_UV_STARTS_AT_TOP
        uv.y = 1.0 - uv.y;
    #endif

    outColor = SAMPLE_TEXTURE2D_X_LOD(_GBuffer2, sampler_PointClamp, uv, 0);
}
```

### _GBuffer

``` cs
//  Make sure this is consistent with DeferredLights.cs
private static readonly int s_GbufferLightingIndex = 3; // _GBuffer3 is the activeColorTexture

private static readonly int[] s_GBufferShaderPropertyIDs = new int[]
{
    Shader.PropertyToID("_GBuffer0"), // Albedo
    Shader.PropertyToID("_GBuffer1"), // Specular Metallic
    Shader.PropertyToID("_GBuffer2"), // Normals and Smoothness      ( _CameraNormalsTexture )
    Shader.PropertyToID("_GBuffer3"), // Lighting
    Shader.PropertyToID("_GBuffer4"), // (optional) Depth            ( _CameraDepthTexture )
    Shader.PropertyToID("_GBuffer5"), // (optional) Rendering Layers ( _CameraRenderingLayersTexture )
    Shader.PropertyToID("_GBuffer6"), // (optional) ShadowMask (Rendering Layers가 없으면 이게 _GBuffer5가 된다)
};
```
