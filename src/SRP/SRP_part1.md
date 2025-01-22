Class Blitter

https://github.com/Unity-Technologies/Graphics/blob/master/Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blitter.cs


Various blit (texture copy) utilities for the Scriptable Render Pipelines.


| 함수 이름                  | 설명                                                 | 사용 사례                                                             |
| -------------------------- | ---------------------------------------------------- | --------------------------------------------------------------------- |
| BlitTexture                | 지정된 텍스처를 다른 텍스처로 복사하는 가장 기본적인 | 간단한 텍스처 복사 작업을 수행할 때                                   |
| BlitCameraTexture          | 카메라의 렌더링 결과를 대상으로 복사하는             | 카메라의 출력 텍스처를 다른 텍스처로 복사할 때                        |
| BlitCameraTexture2D        | 카메라에서 2D 텍스처로 복사하는                      | 2D 스프라이트나 UI에 카메라 출력 텍스처를 사용할 때                   |
| BlitQuad                   | 쿼드 형태로 텍스처를 복사하는                        | 기본적인 텍스처 복사 작업을 수행할 때                                 |
| BlitQuadWithPadding        | 패딩을 추가하여 쿼드 형태로 텍스처를 복사하는        | 텍스처의 가장자리가 부드럽게 보이도록 패딩을 추가할 때                |
| BlitQuadSingleChannel      | 단일 채널 텍스처를 쿼드 형태로 복사하는              | 그레이스케일 텍스처나 알파 채널만 복사할 때                           |
| BlitColorAndDepth          | 색상과 깊이 텍스처를 동시에 복사하는                 | 렌더링 결과에서 색상과 깊이를 동시에 복사할 필요가 있을 때            |
| BlitCubeToOctahedral2DQuad | 큐브 맵 텍스처를 오타헤드럴 2D 쿼드로 변환하는       | 환경 맵을 사용하여 조명 효과를 적용할 때 큐브 맵을 평면으로 변환할 때 |
| BlitOctahedralWithPadding  | 오타헤드럴 텍스처를 패딩과 함께 복사하는             | 텍스처의 가장자리가 부드럽게 보이도록 패딩을 추가할 때                |


