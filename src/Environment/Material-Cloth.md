# Cloth

``` hlsl
float ClothShading( float3 V, float3 N, float RimScale, float RimExp, float InnerScale, float InnerExp )
{
    float fresnel = saturate(dot(V, N));
    float Rim = RimScale * pow(1 - fresnel, RimExp);
    float Inner = InnerScale * pow(fresnel, InnerExp);
    return Rim + Inner;
}
```

## Ref

- [SIGGRAPH2010 - Uncharted 2: Character Lighting and Shading](https://advances.realtimerendering.com/s2010/index.html)
- [2017 - [GDC2017] Technical Artist Bootcamp: Shaders 201: Creating Art with Math](https://gdcvault.com/play/1024282/Technical-Artist-Bootcamp-Shaders-201)
  - <https://youtu.be/wfh3iSkNMOQ?si=8q-Z9pecAyHE0wFN&t=680>
