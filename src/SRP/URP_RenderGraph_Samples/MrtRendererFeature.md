MRT: `M`ultiple `R`ender `T`argets


``` cs
public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    TextureHandle[] handles = new TextureHandle[3];
    for (int i = 0; i < 3; i++)
    {
        handles[i] = renderGraph.ImportTexture(m_RTs[i], m_RTInfos[i]);
    }

    using (IRasterRenderGraphBuilder builder = renderGraph.AddRasterRenderPass("MRT Pass", out PassData passData))
    {
        UniversalResourceData resourceData = frameData.Get<UniversalResourceData>();

        passData.colorTextureHandle = resourceData.activeColorTexture;
        passData.texName = m_texName;
        passData.material = m_Material;


        builder.UseTexture(passData.colorTextureHandle);
        for (int i = 0; i < 3; i++)
        {
            builder.SetRenderAttachment(handles[i], i);
        }
        builder.SetRenderFunc((PassData data, RasterGraphContext rgContext) => ExecutePass(data, rgContext));
    }
}

static void ExecutePass(PassData data, RasterGraphContext rgContext)
{
    data.material.SetTexture(name: data.texName, value: data.colorTextureHandle);
    rgContext.cmd.DrawProcedural(Matrix4x4.identity, data.material, shaderPass: 0, MeshTopology.Triangles, vertexCount: 3);
}
```

``` cs
RTHandle[] m_RTs = new RTHandle[3];
RenderTargetInfo[] m_RTInfos = new RenderTargetInfo[3];

public void Setup(string texName, Material material, RenderTexture[] renderTextures)
{
    m_Material = material;
    m_texName = string.IsNullOrEmpty(texName) ? "_ColorTexture" : texName;

    for (int i = 0; i < 3; i++)
    {
        RenderTexture rednerTexture = renderTextures[i];
        if (m_RTs[i] == null || m_RTs[i].rt != rednerTexture)
        {
            if (m_RTs[i] != null)
            {
                m_RTs[i].Release();
            }

            m_RTs[i] = RTHandles.Alloc(rednerTexture, name: $"ChannelTexture[{i}]");
            m_RTInfos[i] = new RenderTargetInfo
            {
                format = rednerTexture.graphicsFormat,
                height = rednerTexture.height,
                width = rednerTexture.width,
                bindMS = rednerTexture.bindTextureMS,
                volumeDepth = rednerTexture.volumeDepth,
                msaaSamples = 1,
            };
        }
    }
}
```

``` hlsl
Shader "Hidden/MrtColor"
{
    Properties
    {
        _ColorTexture("ColorTexture", 2D) = "white" {}
    }
    SubShader
    {
        Cull Off
        ZWrite Off

        Pass
        {
            HLSLPROGRAM
            #pragma vertex Vert
            #pragma fragment frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.core/ShaderLibrary/GlobalSamplers.hlsl"

            TEXTURE2D_X(_ColorTexture);

            struct Attributes
            {
                uint vertexID : SV_VertexID;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct Varyings
            {
                float4 positionCS : SV_POSITION;
                float2 texcoord   : TEXCOORD0;
                UNITY_VERTEX_OUTPUT_STEREO
            };

            Varyings Vert(Attributes input)
            {
                Varyings output;

                output.positionCS = GetFullScreenTriangleVertexPosition(input.vertexID);
                output.texcoord = GetFullScreenTriangleTexCoord(input.vertexID);

                return output;
            }
    

            // MRT shader
            struct FragmentOutput
            {
                half4 dest0 : SV_Target0;
                half4 dest1 : SV_Target1;
                half4 dest2 : SV_Target2;
            };

            FragmentOutput frag(Varyings input) : SV_Target
            {
                half4 color = SAMPLE_TEXTURE2D_X(_ColorTexture, sampler_LinearRepeat, input.texcoord); 
                FragmentOutput output;
                output.dest0 = half4(color.r, 0.0, 0.0, 1.0);
                output.dest1 = half4(0.0, color.g, 0.0, 1.0);
                output.dest2 = half4(0.0, 0.0, color.b, 1.0);
                return output;
            }
            ENDHLSL
        }
    }
}
```



### Full screen quad vs full screen triangle

- 사각형의 전체화면 그리기 위해
  - (full screen quad) 삼각형 2개로 사각형을 만드는것이 아닌 
  - (full screen triangle) 삼각형을 크게 그려서 삐저나온 부분을 잘라 사각형 그림.

rgContext.cmd.DrawProcedural(Matrix4x4.identity, data.material, shaderPass: 0, MeshTopology.Triangles, vertexCount: 3);

- GPU는 한 번에 하나씩 픽셀을 렌더링하지 않고 2x2에서 8x8 픽셀 사이의 그룹으로 렌더링합니다.
- full screen quad로 그리면 삼각형이 맞닿은 대각선이 겹침
- 겹치는 부분만큼 오버드로우가 생김.

- <https://knarkowicz.wordpress.com/2010/10/26/shader-optimizations/>
- <https://www.saschawillems.de/blog/2016/08/13/vulkan-tutorial-on-rendering-a-fullscreen-quad-without-buffers/>

