# 텍스쳐 포멧

요세는 좋아져서 ASTC쓰면 될듯.

|       |                                       | NPOT |
| ----- | ------------------------------------- | ---- |
| ETC   | Ericsson Texture Compression          | X    |
| PVRTC | PowerVR Texture Compression           | X    |
| ASTC  | Adaptive Scalable Texture Compression | O    |

|           | Graphic Library | Android API | version | 코드명     | Linear지원 |
| --------- | --------------- | ----------- | ------- | ---------- | ---------- |
| ETC1      | es2.0           | 8           | 2.2.x   | Froyo      | x          |
| ETC2 /EAC | es3.0           | 18          | 4.3.x   | Jelly Bean | O          |
| ASTC      | es3.1+AEP       | 21          | 5.0     | Lollipop   | O          |

|      |              |                       |
| ---- | ------------ | --------------------- |
| ASTC | A8 processor | iPhone 6, iPad mini 4 |

|      |             |          |
| ---- | ----------- | -------- |
| 2015 | iPad mini 4 | Apple A8 |
| 2014 | iPhone6     | Apple A8 |

## 텍스쳐 파일

|       |                                            |
| ----- | ------------------------------------------ |
| etc   | Eric Texture Compression                   |
| etc2  | Eric Texture Compression2                  |
| PVRTC | PowerVR Texture Compression                |
| ASTC  | Adaptive Scalable Texture Compression      |
| DDS   | DirectDraw Surface                         |
| bmp   | Microsoft Window Device Independent Bitmap |
| jpeg  | Joint Photographic Experts Group           |
| png   | Portable Network Graphic                   |
| gif   | Graphic Interchange Format                 |
| tga   | Truevision Graphics Adapter                |
| webp  |                                            |
| webp2 |                                            |
| avif  |                                            |
| bpg   | Better Portable Grpahics - 특허걸려있음    |

- [Display color와 Digital texture format의 이해](https://www.slideshare.net/illustor/display-color-digital-texture-format)

## Ref

- <https://en.wikipedia.org/wiki/Ericsson_Texture_Compression>
- <https://en.wikipedia.org/wiki/Adaptive_Scalable_Texture_Compression>
- <https://en.wikipedia.org/wiki/PVRTC>
- <https://gametorrahod.com/pvrtc-vs-astc-texture-compression-on-an-ios-device/>
