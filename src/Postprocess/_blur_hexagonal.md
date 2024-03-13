# Hexagonal Blur

- [Five Rendering Ideas from Battlefield 3 & Need For Speed: The Run from DICE](https://www.ea.com/frostbite/news/more-performance-five-rendering-ideas-from-battlefield-3-and-need-for-speed-the-run)
  - [pdf(코멘트 포함)](http://advances.realtimerendering.com/s2011/White,%20BarreBrisebois-%20Rendering%20in%20BF3%20(Siggraph%202011%20Advances%20in%20Real-Time%20Rendering%20Course).pdf)
  - https://github.com/zigguratvertigo/HexBokehBlur

3pass(basic)
    1 직각  (Vertical) :  90( PI/2)
    2 대각  (Diagonal) : -30(-PI/6)
        - 다음 패스로 넘겨주기전에 직각과 합친다
    3 마름모(Rhomboid)
        - Coc회전이 필요하면 여기서
        - 1에서 온것을 -30(-PI/6)      - 마름모의 좌상단         체우기
        - 2에서 온것을 -150(5 * -PI/6) - 마름모의 우상단 및 하단 체우기
        

2pass(basic)
    1 MRT : 직각(Vertical)COLOR0 + 직각대각(Vertical + Diagonal)COLOR1
    2 마름모(Rhomboid)
        - Coc회전이 필요하면 여기서
        - 직각을     -30(-PI/6)로
        - 직각대각을 -150(5 * -PI/6)로



5 pass(iterative refinement) : 
    [Crysis Next-Gen Effects (GDC 2008) - Tiago Sousa](https://www.slideshare.net/TiagoAlexSousa/crysis-nextgen-effects-gdc-2008)
    1 직각, 대각
    2 직각, 직각 + 대각
    3 대각 - 좌측하단(좌측상단         완성)
    4 대각 - 우측하단(우측상단 및 하단 완성)
    5 합체

``` hlsl
// === 강조부분 걸러내기
float3 scene = tex2D(sceneTexture, uv).rgb;
scene       *= dot(float3(0.3f, 0.59f, 0.11f), scene); // grayscale을 구하고
scene       *= scene;                                  // 제곱하여 명암비를 높이고
scene       *= 1500;                                   // 곱해주어 수치를 높인다


// ==== 다운샘플링
// CoC : circle of confusion, 착란원(조리개의 형태에 기반한)

// ==== 직각 + 대각

// ==== 마름모
```


```
const int NUM_SAMPLES = 16;

float4 BlurTexture(sampler2D tex, float2 uv, float2 direction)
{
   float4 finalColor = 0.0f;
   float  blurAmount = 0.0f;

   // uv에 오프셋을 더해줘야 한다(안그러면 마름모가 살작 겹쳐 경계부분이 밝게 보인다)
   uv += direction * 0.5f;
   
   for (int i = 0; i < NUM_SAMPLES; ++i)
   {
      float4 color = tex2D(tex, uv + direction * i);
      color       *= color.a;
      blurAmount  += color.a;   
      finalColor  += color;
   }
  
   return (finalColor / blurAmount);
}
```



https://mynameismjp.wordpress.com/2011/02/28/bokeh/