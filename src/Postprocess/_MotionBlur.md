# MotionBlur

- Frame Blur
- Position Reconstruction
- Velocity Buffer


- <https://blog.naver.com/sorkelf/40158984437>
- <https://medium.com/spaceapetech/motion-blur-for-mobile-devices-in-unity-656c8047508>




[2023 - [GDC2023] Stupid OpenGL Shader Tricks by Simon Green, NVIDIA](https://www.nvidia.com/docs/io/8230/gdc2003_openglshadertricks.pdf)
Image space (2.5D) motion blur
- 3 stages:
  - Render scene to texture
    - At current time
  - Calculate velocity at each pixel
    - Using vertex shader
    - Calculate current position – previous position
  - Render motion blurred scene
    - Using fragment shader
    - Look up into scene texture

``` hlsl
// Calculate velocity at each pixel
struct a2v {
    float4 coord;
    float4 prevCoord;
    float3 normal;
    float2 texture;
};
struct v2f {
    float4 hpos : HPOS;
    float3 velocity : TEX0;
};

v2f main(a2v in,
    uniform float4x4 modelView,
    uniform float4x4 prevModelView,
    uniform float4x4 modelViewProj,
    uniform float4x4 prevModelViewProj,
    uniform float3 halfWinSize,
)
{
    v2f out;
    
    // transform previous and current pos to eye space
    float4 P = mul(modelView, in.coord);
    float4 Pprev = mul(prevModelView, in.prevCoord);

    // transform normal to eye space
    float3 N = vecMul(modelView, in.normal);
    
    // calculate eye space motion vector
    float3 motionVector = P.xyz - Pprev.xyz;
    
    // calculate clip space motion vector
    P = mul(modelViewProj, in.coord);
    Pprev = mul(prevModelViewProj, in.prevCoord);
    
    // choose previous or current position based
    // on dot product between motion vector and normal
    float flag = dot(motionVector, N) > 0;
    float4 Pstretch = flag ? P : Pprev;
    out.hpos = Pstretch;
    
    // do divide by W -> NDC coordinates
    P.xyz = P.xyz / P.w;
    Pprev.xyz = Pprev.xyz / Pprev.w;
    Pstretch.xyz = Pstretch.xyz / Pstretch.w;
    
    // calculate window space velocity
    float3 dP = halfWinSize.xyz * (P.xyz - Pprev.xyz);
    out.velocity = dP;
    return v2f;
}
```


``` hlsl
// Motion Blur Shader Code
struct v2f {
    float4 wpos : WPOS;
    float3 velocity : TEX0;
};
struct f2f {
    float4 col;
};

f2fConnector main(
    v2f in,
    uniform samplerRECT sceneTex,
    uniform float blurScale = 1.0
)
{
    f2f out;
    
    // read velocity from texture coordinate
    half2 velocity = v2f.velocity.xy * blurScale;
    
    // sample scene texture along direction of motion
    const float samples = SAMPLES;
    const float w = 1.0 / samples; // sample weight
    fixed4 a = 0; // accumulator
    float i;
    for(i = 0; i < samples; i += 1)
    {
        float t = i / (samples-1);
        a = a + x4texRECT(sceneTex, in.wpos + velocity*t) * w;
    }
    out.col = a;
}
```

