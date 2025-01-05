- 프레임버퍼 페치는비디오 메모리 대신 GPU의 온칩 메모리에서 프레임버퍼에 액세스할 수 있는 렌더 패스를 허용하는 프로세스
  - 모바일 기기에서 타일 기반 지연 렌더링(TBDR)을 사용하는 모바일 기기에서 렌더링 속도가 향상

https://docs.unity3d.com/6000.0/Documentation/Manual/urp/render-graph-framebuffer-fetch.html

FBF( Frame Buffer Fetch)

# `SetInputAttachment`

- support API
  - Vulkan
  - Metal

``` hlsl
Input
    FRAMEBUFFER_INPUT_X_HALF
    FRAMEBUFFER_INPUT_X_FLOAT
    FRAMEBUFFER_INPUT_X_INT
    FRAMEBUFFER_INPUT_X_UINT

Load
    LOAD_FRAMEBUFFER_INPUT(idx)
    LOAD_FRAMEBUFFER_INPUT_MS(idx,sampleIdx)  // 멀티샘플링(MSAA)이 활성화된 텍스처를 처리하기 위해 사용됨.
```


``` cs
builder.SetInputAttachment(sourceTextureHandle, 0, AccessFlags.Read);
```

``` cs
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

    TextureHandle source = resourceData.activeColorTexture;

    TextureDesc destinationDesc = renderGraph.GetTextureDesc(source);
    destinationDesc.name = "FBFetchDestTexture";
    destinationDesc.clearBuffer = false;

    if (destinationDesc.msaaSamples == MSAASamples.None || RenderGraphUtils.CanAddCopyPassMSAA())
    {
        bool isUseMSAA = destinationDesc.msaaSamples != MSAASamples.None;

        TextureHandle fbFetchDestination = renderGraph.CreateTexture(destinationDesc);

        FBFetchPass(renderGraph, frameData, srcHandle: source, dstHandle: fbFetchDestination, isUseMSAA);

        renderGraph.AddCopyPass(fbFetchDestination, source, passName: "Copy Back FF Destination (also using FBF)");
    }
    else
    {
        Debug.Log("Can't add the FBF pass and the copy pass due to MSAA");
    }
}

private void FBFetchPass(RenderGraph renderGraph, ContextContainer frameData, TextureHandle srcHandle, TextureHandle dstHandle, bool isUseMSAA)
{
    string passName = "FrameBufferFetchPass";

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass(passName, out PassData passData))
    {
        passData.material = m_FBFetchMaterial;
        passData.isUseMSAA = isUseMSAA;

        builder.SetInputAttachment(tex: srcHandle, index: 0, AccessFlags.Read);
        builder.SetRenderAttachment(tex: dstHandle, index: 0); // Output
        builder.AllowPassCulling(false);
        builder.SetRenderFunc((PassData data, RasterGraphContext context) => ExecuteFBFetchPass(data, context));
    }
}

static void ExecuteFBFetchPass(PassData data, RasterGraphContext context)
{
    context.cmd.DrawProcedural(Matrix4x4.identity, data.material, data.useMSAA ? 1 : 0, MeshTopology.Triangles, 3, 1, null);
}
```


``` hlsl
Shader "FrameBufferFetch"
{
   SubShader
   {
       Tags { "RenderType"="Opaque" "RenderPipeline" = "UniversalPipeline"}
       ZWrite Off Cull Off
       Pass
       {
           Name "FrameBufferFetch"

           HLSLPROGRAM
           #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
           #include "Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl"

           #pragma vertex Vert
           #pragma fragment Frag

           FRAMEBUFFER_INPUT_HALF(0);

           // Out frag function takes as input a struct that contains the screen space coordinate we are going to use to sample our texture. It also writes to SV_Target0, this has to match the index set in the UseTextureFragment(sourceTexture, 0, …) we defined in our render pass script.   
           float4 Frag(Varyings input) : SV_Target0
           {
               UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input); // for XR platform

               float2 uv = input.texcoord.xy;
               half4 color = LOAD_FRAMEBUFFER_INPUT(0, input.positionCS.xy);

               return half4(0,0,1,1) * color;
           }

           ENDHLSL
       }

       Tags { "RenderType"="Opaque" "RenderPipeline" = "UniversalPipeline"}
       ZWrite Off Cull Off
       Pass
       {
           Name "FrameBufferFetchMS"

           HLSLPROGRAM
           #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
           #include "Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl"

           #pragma vertex Vert
           #pragma fragment Frag
           #pragma target 4.5
           #pragma require msaatex

           FRAMEBUFFER_INPUT_HALF_MS(0);

           // Out frag function takes as input a struct that contains the screen space coordinate we are going to use to sample our texture. It also writes to SV_Target0, this has to match the index set in the UseTextureFragment(sourceTexture, 0, …) we defined in our render pass script.   
           float4 Frag(Varyings input, uint sampleID : SV_SampleIndex) : SV_Target0
           {
               UNITY_SETUP_STEREO_EYE_INDEX_POST_VERTEX(input); // for XR platform

               float2 uv = input.texcoord.xy;
               half4 color = LOAD_FRAMEBUFFER_INPUT_MS(0, sampleID, input.positionCS.xy);
               
               // Modify the sampled color
               return half4(0,0,1,1) * color;
           }

           ENDHLSL
       }
   }
}
```