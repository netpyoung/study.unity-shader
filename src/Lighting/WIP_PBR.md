# PBR

- PBR(Physical based rendering) / PBS(Physical based shader)

Energy = diffuse + specular + transmission

- https://renderwonk.com/publications/
- 
[[Ndc13]Ndc 2013 김동석:UDK로 물리기반 셰이더 만들기](https://www.slideshare.net/jalnaga/ndc13ndc-2013-udk-19999169)


siggraph 2010
tri-Ace
Practical Implementation of Physically-Based Shading Models at tri-Ace (Yoshiharu Gotanda) 
[slide](https://renderwonk.com/publications/s2010-shading-course/gotanda/slide_practical_implementation_at_triace.pdf)
[cource notes](https://renderwonk.com/publications/s2010-shading-course/gotanda/course_note_practical_implementation_at_triace.pdf)

siggraph 2011
Lazarov
Physically Based Lighting in Call of Duty: Black Ops
Dimitar Lazarov, Lead Graphics Engineer, Treyarch
 
 Sébastien Lagarde
Moving Frostbite to PBR (Sébastien Lagarde & Charles de Rousiers) 
https://blog.selfshadow.com/publications/s2014-shading-course/frostbite/s2014_pbs_frostbite_slides.pdf


- Etc
  - BSDF(Bidirectional Scattering Distribution Function)
  - BTDF(Bidirectional Transmission Distribution Function)
  - BSSRDF(Bidirectional Scattering Surface Reflectance Distribution Function)
  - SPDF(Scattering Probability Density Function)

## Custom PBR

TODO

- https://www.slideshare.net/dongminpark71/ndc19-pbr-143928930  
- [언차티드4 테크아트 파트4 Special Case Materials - Moss & Wetness & Glass](https://www.slideshare.net/DaeHyekKIM/4-4-special-case-materials-moss-wetness-glass)
- [ 언차티드4 테크아트 파트3 MicroShadowBRDF Term ](https://www.slideshare.net/DaeHyekKIM/4-3-microshadowbrdf-term)
  
## Ref

- <https://www.slideshare.net/MRESC/pbr-vol2-131205432>
- Adobe The PBR Guide
  - <https://substance3d.adobe.com/tutorials/courses/the-pbr-guide-part-1>
  - <https://substance3d.adobe.com/tutorials/courses/the-pbr-guide-part-2>

https://leegoonz.blog/2020/01/05/energy-conserved-specular-blinn-phong/
https://www.rorydriscoll.com/2009/01/25/energy-conservation-in-games/




## 재질
- https://dev.epicgames.com/documentation/en-us/unreal-engine/physically-based-materials-in-unreal-engine
- https://creativecloud.adobe.com/learn/substance-3d-designer/web/the-pbr-guide-part-1
- https://creativecloud.adobe.com/learn/substance-3d-designer/web/the-pbr-guide-part-2

## custom
난반사(Diffuse Reflection)
정반사(Specluar Reflection)

|        | 정반사 | 난반사 |
| ------ | ------ | ------ |
| 비금속 | 흰색   | 기본색 |
| 금속   | 기본색 | 흰색   |


- 방식
  - ref
    - [물리기반 머터리얼 상식](https://youtu.be/1biT79BtSkw)
      - 직접광보다 간접광(Environment map)이 중요.
        - https://marmoset.co/shop/
          - Perpetual License $31900one-time fee (USD)
    - [디퓨즈와 스페큘러](https://youtu.be/T6-OnODMgdE)
      - [How To Split Specular And Diffuse In Real Images](http://filmicworlds.com/blog/how-to-split-specular-and-diffuse-in-real-images/)
      - https://lifeisforu.tistory.com/382
      - https://www.virial.com/reflection-models.html
  - metal - roughness
    - base color : brdf color
    - metallic : reflectance ( specular level, Index of Refelection - 별명이 artistic metallic - 1.6)
    - roughness: glossiness를 선형화 시켜서 뒤짚은 값.
      - 사람은 대략 0.5
  - specular / glossiness
    - diffuse(albedo (알비도)) / specular / glossiness

base color/metalic / roughness 방식
- 단점
  - 텍셀 밀도가 낮을시 metalic edge 현상 발생
- base color
  - PBR Safe Color
    - https://helpx.adobe.com/substance-3d-designer/substance-compositing-graphs/nodes-reference-for-substance-compositing-graphs/node  -library/material-filters/pbr-utilities/pbr-albedo-safe-color.html
    - https://helpx.adobe.com/substance-3d-designer/substance-compositing-graphs/nodes-reference-for-substance-compositing-graphs/node-library/material-filters/pbr-utilities/pbr-basecolor-metallic-validate.html
- 메탈릭
  - albedo : 밝아야함(생각보다 어둡게 나오는 경우가 많음)
  - 금속/비금속을 나누는 기준임으로 어중간한 값들의 사용은 자제해야한다.
- 거칠기 : 높을수록 정반사 비율이 낮아짐.
  - 기울여 봐야 Fresnel의 차이를 확인 할 수 있음.

