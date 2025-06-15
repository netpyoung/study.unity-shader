# Trim sheet



``` txt
배경용 건물을 작업하다 보면 그림처럼 기둥의 밑단을 작업해 줍니다. 그런데 이런 작업물의 텍스쳐는 필연적으로 가로로 긴 직사각형이 됩니다. 게임용 텍스쳐는 가능한한 정사각형이 되는게 좋으므로 이렇게 가로로 긴 텍스쳐를 따로 모아 아래처럼 만들어 줍니다.
    - 환상정원 https://blog.naver.com/blue9954/221636740320
```

- [Sci Fi Stair Generator | Part 3 | Unwrap UVs for Trim Sheet](https://www.youtube.com/watch?v=cZYVdJLaDek)
- [The Different Types Of Trim Sheets & How They Are Used In Games (Part 1)](https://www.youtube.com/watch?v=Y4cZbi8Mvxg)

- 트림시트 나눌 계획세우기
  - 어느부분을 얼마만큼 할당할지. 큼직큼직한곳, 디테일한곳
  - 아무것도 할당하지 않은 예비영역도 넣어두기

## Torch3DAcademy

- [유투브 채널 @Torch3DAcademy](https://www.youtube.com/@Torch3DAcademy/)
  - [TrimSheet Workflow - Simple SF box](https://www.youtube.com/playlist?list=PLSFzUDFhjXd4HwJU5n9PQQfnIsWYpSf_J)
  - [TrimSheet Workflow - MedievalHouse](https://www.youtube.com/playlist?list=PLSFzUDFhjXd7AfreG9nhhY3in29F_VklR)

- AO는 버텍스 칼라의 Alpha채널 활용 가능


### TrimSheet Workflow - Simple SF box

- 트림시트 나누고
  - [트림시트 워크플로우 6강[트림시트 실전편 - 트림시트 제작]](https://www.youtube.com/watch?v=PwLgevQw6FM)
- 모서리 벗겨진 효과(엣지웨어)
  - [트림시트 워크플로우 10강[트림시트 실전편 - 엣지웨어 효과 제작]](https://www.youtube.com/watch?v=2qxof_B8Mdo)
- 데칼 시트 자체에서 
  - [트림시트 워크플로우 11강[트림시트 실전편 - 데칼시트로 꾸미기 + 버텍스 컬러링]](https://www.youtube.com/watch?v=viRjuVgl31k)
- 디테일
  - [트림시트 워크플로우 12강[트림시트 실전편 완결! - 엔진 임포트, 셰이더 제작, 그리고 완성]](https://www.youtube.com/watch?v=aa34tPAh_XQ)
  - 버텍스 alpha채널에 AO배이크
  - ORM에 Alpha에 culvature
  ``` hlsl
  EdgeWear = 
    saturate(
        saturate(
            (culvature - 0.5 * _EdgeWearWidth)
            - (_EdgeWearWidth * 0.5 - 1)
        )
        - (noise(_NoiseTiling) * _EdgeWearIntensity)
        * _EdgeWearHardness
      );
  ```
  - 셰이더에서
    - 기본컬러: 베이스텍스쳐 + EdgeWear
    - metalic : 메탈릭텍스쳐 + EdgeWear
    - roughness: roughness - EdgeWear

  ### TrimSheet Workflow - MedievalHouse




- 나무질감 , 엣지 데미지
  - Roughness : Culvature에서 GradientMap으로 반전시키고 기본 색을 GrayScale / Level시킨것을 빼주기
  - [트림시트 중급편 - 중세 집 제작 3강[나무트림시트]](https://www.youtube.com/watch?v=sIsOk2-ie8Q)
- 흠집내기 - 찍힌자국, 실선, 얼룩
  - Cell이랑 타일 이용해서 찍힌자국
  - Cell 이용해서 얼룩
  - Scratches Generator로 스크레치
  - [[3강 선수강 필수]트림시트 중급편 - 중세 집 제작 4강[나무트림시트 재활용한 메탈트림시트 제작]](https://www.youtube.com/watch?v=DuMt_jMMG2M)
  - [[3강 선수강 필수]트림시트 중급편 - 중세 집 제작 5강[돌 트림시트 제작]](https://www.youtube.com/watch?v=DpOIaHyloww)


## Ref

- [What are Trim Sheets?](https://youtu.be/uUJShalzWy8?si=izKJ28MLf0ZXhIUH)
- [The Ultimate Trim Texturing Techniques](https://gdcvault.com/play/1022324/The-Ultimate-Trim-Texturing-Techniques)
- [Ultimate Trim Generator for Substance Designer Overview](https://youtu.be/-zWdwo-azfA?si=CFKJq2gTdgwMlWWo)
- 안콜3D
  - [퀄리티와 최적화를 모두 잡는 라오어의 트림기술](https://youtu.be/QwgxX0pB-Bo?si=GGWJul0HjY9ht-Pd)
  - [진짜 기발한 텍스쳐링 방식 (+ 쉐이더) | 전천후 배경모델러 메가리기 [5부]](https://youtu.be/oLV7nExQ0AY?si=uYpPfOxQZm7De2tG)
  - [배경 작업자만이 하는 일.](https://youtu.be/KX6CWEvlmJ4?si=GZuU-PIEEdN38LOb)
  - <https://www.artstation.com/blogs/jennifermcgarry/yd4Q/jenns-guide-to-trim-sheets>
  - <https://www.exp-points.com/vuk-single-material-modular-kit-environment-ue4>
  - <https://www.exp-points.com/king-arthur-grandiose-kings-chamber>
- <https://www.patreon.com/posts/lets-talk-about-94999921>








