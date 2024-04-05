# Pipeline

텍스쳐

메쉬
    VertexBuffer
        pos/uv/normal/color
    IndexBuffer

Gamma / Linear Pipeline (HDR)

Rendering
    Forward 
    Forward Plus
    Deferred


CPU =DrawCall=> GPU

Culling

Input Assembly
    Shader:Vertex
        object/world/camera/clip
    Shader:Hull
    Shader:Tesselator
    Shader:Domain
    Shader:Geometry
    
    Rasterizer
        정점간 보간
    Early Depth Testing

    Shader:Pixel

    Depth Testing
Render Target Output

포스트프로세스
(HDR)Luminance
(HDR)Bloom
(HDR)Eye Adaptation
(HDR)Tone Mapping

SSR
SSAO
SSGI
Motion Blur

Depth of Field - Bokeh(geometry / gather)
Color Filtering
Gamma Control

AA



LOD
HLOD
Lumen
