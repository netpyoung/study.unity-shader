- https://scahp.tistory.com/95
- Dual-depth relief interior mapping
  - ex) UE5 CitySample 건물 내부

|                           |                                       |
| ------------------------- | ------------------------------------- |
| Interior mapping          | 공간을 렌더링 하기 위해               |
| Dual-depth relief mapping | 공간 내부의 가구들을 렌더링 하기 위해 |


- Relief mapping 의 구현은 텍스쳐 공간(탄젠트 공간)에서의 Ray marching 을 사용
- Ray marching
  - Single-depth relief mapping
    - Linear tracing 
    - Binary tracing
  - Dual-depth relief mapping
    - Single-depth relief mapping 에서 발생하는 Skin 이라 불리는 Artifact를 완화(후면의 디테일이 뭉개지는)
    - 앞면과 뒷면에 대한 Depth texture를 모두 생성.
      - 그리고 이 두 개의 depth texture를 사용하여 레이마칭을 수행