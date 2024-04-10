- Graphics Pipeline and Optimization for Artists - Part 5
  - Performance and Optimization in Unity
  - https://www.youtube.com/watch?v=zMgtl8_PmXE
- [Batching techniques in Unity](https://cmwdexint.com/2018/06/19/batching-techniques-in-unity/)



- drawcall
  - 메쉬 하나를 그릴때
  - 무엇을, 어떻게 그릴까라는 것을 명령하는 것

- DrawCall 발생조건
  - 기본조건: 1메쉬 + 1머티리얼
  - N 오브젝트 + M머티리얼(유니크)

``` txt

CPU                                                       | GPU
Application = DrawCall=> Graphics API => Graphics Driver =|=> RenderStateTable => Rendering

```

- Batch
  - Draw Call + RenderState Change
- Batching
  - 1 batch for many objects
- SetPass Call (= material change)
  - RenderState Change 를 발생시킴
  - 머티리얼 변경요소
    - shader parameter: texture, color ...
    - pipeline settings: alpha blending, z testing ...
- Built-in Batcher(Legacy)
  - 머티리얼 기준
- SRP Batcher
  - 셰이더 기준
  - 동일 셰이더를 머티리얼들이 서로 공유하면 배칭이됨
    - 머티리얼 파라미터가 다르거나
    - 메쉬가 달라도
    - 셰이더만 같다면 배칭이 됨
  - 메쉬만 지원(파티클은 지원하지 않음)
  - 움직이는 메쉬들에도 대응
  - 머티리얼이 많고, 셰이더가 상대적으로 적은 경우에 유리
- GPU Instancing
  - 동일한 메쉬 + 동일한 머티리얼
    - 복사된 동일한 메쉬들을 instances라 부름
    - SkinnedMeshRenderers는 지원 안함
  - SRP Batcher켜지면 우선순위가 낮아서 GPU Instancing는 비활성됨


가속도계
iOS only // Project Settings > Player > Accelerometer Frequency : 기본값(60 Hz) => 적거나 혹은 끄거나