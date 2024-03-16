서브스텐스 디자이너 - 15만원정도함 년마다 나오니 1월에 이전버전 세일 노리는것도

[udemy: The Ultimate 2D & 3D Shader Graph VFX Unity Course](https://www.udemy.com/course/the-ultimate-2d-3d-shader-graph-vfx-unity-course/)


## 2D Fire Projectile Shader Graph

심리스텍스쳐제작(포토샵) 베이스텍스쳐에 사각형으로 심리스 베이스를 잡음

불꽃 진행방향이 좀 특이했음

진행방향 왼쪽
불꽃방향 왼쪽
디솔브방향 오른쪽

3겹으로 쌓아올리니 그럴싸함(코어/주변/연기)



## 2D Wind Vegetation Shader Graph (Vertex Movement Tutorial)
전체이미지를 움직이는게 아닌 uv를의 v를 상단 부분의 마스크로 이용

## 2D Dissolve Shader Graph
A : 노이즈에 스탭을 줘서 알파값으로 디솔브
B : 노이즈 스탭값을 추가한것
C : 아웃라인시킨것(B - A)
[ColorUsage(true, true)] 어트리뷰트로 intensity 적용가능한 색상편집다이얼로그

## 2D Pixel Art Shader Graphs
픽셀라이즈된 이미지를 시간에 따른 노이즈로 흩뜨린것으로 나뭇가지 바람에 날리는 효과(2D)

## 3D Magical Portal
float2 Center      = 0.5;
float  RadialScale = 1;
float  LengthScale = 1;
float2 delta = UV - Center;
float radius = length(delta) * 2 * RadialScale;
float angle = atan2(delta.x, delta.y) * 1.0/6.28 * LengthScale;
float2 polarCoord = float2(radius, angle);

polarCoord.x : 원
polarCoord.y : 우측

outerCircle = smoothstep(_CircleClip , _CircleClip  + _Feather, polarCoord.x);
edge1 = _CircleClip - _RingWidth;
edge2 = edge1 + _Feather;
innerCircle = smoothstep(edge1, edge2, polarCoord.x);
ring = innerCircle - outerCircle

- polarCoord를 적용하기 위해 포토샵으로 텍스쳐 제작
  - 일반 노이즈에 polarCoord를 적용하면 씸리스가 안됨.

포토샵
Filter> Render> Clouds
Filter> Blur> Motion Blur(angle:45, distance:100)
Filter> Other> Offset을 조정하면서 접합부가 보이면 뭉대겨서 접합부를 안보이게 만들어 줘야함.


## 3D Stylized Water Shader - 중요

depthFade = saturate((SceneDepth(Eye) - ScreenPosition(Raw).a) / _Distance);
waterGradientColor = lerp(_SurfaceColor, _DeepColor, depthFade);

noiseNormal = GradientNoise를 반대방향으로 2개를 만들고,  NormalFromHeight로 노말을 구해 노말을 블렌드한다
waterNormalStrength = lerp(0, _WaterNormalStrength, depthFade);
waterNormal = saturate(NormalStrength(noiseNormal, waterNormalStrength));
waterSceneColor = SceneColor((noiseNormal  * 0.1 * _RefractionStrength) + ScreenPosition(Default))
waterColor = lerp(waterSceneColor, waterGradientColor, waterGradientColor.a); 

폼은
waterFormNoise = GradientNoise를 움직이고
waterFormNoiseStep = step(waterFormNoise, depthFade * 2)
waterFormColor = lerp(waterGradientColor, _FormColor, waterFormNoiseStep);
finalWaterColor = lerp(waterColor, waterFormColor, waterFormColor.a);

## Sub-Surface Scattering (SSS) Shader Graph
NdotV 로 림을 만들고 LightIntensity를 곱해 SSS를 만들고
(1 - HeightMap)과 곱해 메인칼라에 더해준다

blender를 이용 Thinknessmap
- UV Editing Tab
  Image> New> Thickness로 이름
- shading tab
노드들 지우고
AmbientOcclusion(Inside 체크 및 Distance 조정)
ImageTexture 노드에 앞서만든 Thickness 선택

- Render Tab
Bake Type> Emit
Max Ray Distance> 0.2
Bake버튼
- UV Editing Tab
  Image> Save As> Thickness.png

## Triplanar Shader Graph

텍스쳐를 입히는것이 오브젝트공간이 아닌 월드공간으로 지형에 어울리는것을 배치할때 자연스럽게

[assetstore:Valley Gray Cliff(무료)](https://assetstore.unity.com/packages/3d/props/exterior/valley-gray-cliff-203730)

height = PositionWS.y
heightSmooth = smoothstep(_2ndTextureHeightBlend - 1, _2ndTextureHeightBlend + 1, height)
heightBlendVal = smoothstep(0, 1 - _2ndTextureThreshold, heightSmooth * normalWS.y)
color = lerp(_MainTex, _2ndTex, heightBlendVal)


Extra content
void Unity_RadialShear_float(float2 UV, float2 Center, float Strength, float2 Offset, out float2 Out)
{
    float2 delta = UV - Center;
    float delta2 = dot(delta.xy, delta.xy);
    float2 delta_offset = delta2 * Strength;
    Out = UV + float2(delta.y, -delta.x) * delta_offset + Offset;
}

51. Colors Theory (Choose the most pleasing color combinations of color) 중요
Shade : add dark
Tint : add White
Tone : add Gray

MonoChromatic 하나
Complementary 하나 그리고 반대
splitcomplementary 하나 그리고 반대편 양쪽 2개
Triadic             3등분
Analogous 주변색
TetradicColors 양쪽2개 반대편 양쪽 2개


== 
- 하나의 Dominant(지배적인) 색 선택
- 되도록 적은 색 선택
https://color.adobe.com/



Oh, Good. That approval came out quickly than what I expacted.
Yes, that address is correct.

And Here is additional information about postal code and phone number.

  Postal code: 05028
  Phone number: +82)010-9315-2848
  
Okay I also search my room to enter Japan on January as possible.

Thank you.