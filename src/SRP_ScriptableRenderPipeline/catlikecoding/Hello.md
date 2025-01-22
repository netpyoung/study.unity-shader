https://catlikecoding.com/unity/tutorials/custom-srp/
- Custom Render Pipeline
- Draw Calls
- Directional Lights
- Directional Shadows
- Baked Light
- Shadow Masks
- LOD and Reflections
- Complex Maps
- Point and Spot Lights
- Point and Spot Shadows
- Post Processing
- HDR
- Color Grading
- Multiple Cameras
- Particles
- Render Scale
- FXAA

https://catlikecoding.com/unity/custom-srp/
- 1.0.0 Modernization
- 2.0.0 Render Graph
- 2.1.0 Renderer Lists
- 2.2.0 Camera Textures
- 2.3.0 Shadow Textures
- 2.4.0 Post FX Passes
- 2.5.0 Structured Buffers
- 3.0.0 Simple Tiled Forward+
- 3.1.0 Forward+ Settings
- 3.2.0 Simplification
- 4.0.0 Unity 6

==

# [Custom Render Pipeline](https://catlikecoding.com/unity/tutorials/custom-srp/custom-render-pipeline/)

RenderPipelineAsset 에셋을 상속받아 RenderPipelineAsset 를 만든다
[CreateAssetMenu]을 써서 메뉴로 만들 수 있게 한다.

RenderPipeline 을 상속받아 RenderPipeline 을 작성

CameraRenderer을 정의하여 카메라 각각의 렌더링을 담당


셰이더에 쓰일 mvp메트릭스 초기화
    scriptableRenderContext.SetupCameraProperties(camera);
스카이박스 그리기
    scriptableRenderContext.DrawSkybox(camera); 

디버거를 위한 샘플(프로파일러와 프레임 디버거에 표시)
    commandBuffer.BeginSample(bufferName);
    commandBuffer.EndSample(bufferName);

렌더 타겟 클리어
    commandBuffer.ClearRenderTarget(clearDepth: true, clearColor: true, backgroundColor: Color.clear);

카메라 컬링
    camera.TryGetCullingParameters(out ScriptableCullingParameters p)
    CullingResults cullingResults = scriptableRenderContext.Cull(ref p);


오브젝트 그리기
    // 불투명
    ShaderTagId shaderTagId = new ShaderTagId("SRPDefaultUnlit");
    SortingSettings sortingSettings = new SortingSettings(camera) {
        criteria = SortingCriteria.CommonOpaque
    };
    DrawingSettings drawingSettings = new DrawingSettings(shaderTagId, sortingSettings);
    FilteringSettings filteringSettings = new FilteringSettings(RenderQueueRange.opaque);
    scriptableRenderContext.DrawRenderers(cullingResults, ref drawingSettings, ref filteringSettings);

    // 중간에 스카이박스
    
    // 투명
    sortingSettings.criteria = SortingCriteria.CommonTransparent;
    drawingSettings.sortingSettings = sortingSettings;
    filteringSettings.renderQueueRange = RenderQueueRange.transparent;
    scriptableRenderContext.DrawRenderers(cullingResults, ref drawingSettings, ref filteringSettings);



에러
    ShaderTagId[] legacyShaderTagIds = {
		new ShaderTagId("Always"),
		new ShaderTagId("ForwardBase"),
		new ShaderTagId("PrepassBase"),
		new ShaderTagId("Vertex"),
		new ShaderTagId("VertexLMRGBM"),
		new ShaderTagId("VertexLM")
	};
    Material errorMaterial = new Material(Shader.Find("Hidden/InternalErrorShader"));
    
    DrawingSettings drawingSettings;
    {
        drawingSettings = new DrawingSettings(legacyShaderTagIds[0], new SortingSettings(camera)) {
            overrideMaterial = errorMaterial
        };
        for (int i = 1; i < legacyShaderTagIds.Length; i++)
        {
            drawingSettings.SetShaderPassName(i, legacyShaderTagIds[i]);
        }
    } 

    FilteringSettings filteringSettings = FilteringSettings.defaultValue;
    scriptableRenderContext.DrawRenderers(cullingResults, ref drawingSettings, ref filteringSettings);

Gizmo
    void DrawGizmos () {
		if (!Handles.ShouldRenderGizmos())
        {
            return;
        }
        scriptableRenderContext.DrawGizmos(camera, GizmoSubset.PreImageEffects);
        scriptableRenderContext.DrawGizmos(camera, GizmoSubset.PostImageEffects);
	}

Unity UI 그리기
    if (camera.cameraType == CameraType.SceneView)
    {
        ScriptableRenderContext.EmitWorldGeometryForSceneView(camera);
    }


클리어 플래그
// BreadcrumbsUnityCsReference/Runtime/Export/Graphics/GraphicsEnums.cs
public enum CameraClearFlags
{
    Skybox = 1,
    Color = 2,
    SolidColor = 2,
    Depth = 3,
    Nothing = 4
}

CameraClearFlags cameraClearflags = camera.clearFlags;
isClearDepth = flags <= CameraClearFlags.Depth;
isClearColor = flags == CameraClearFlags.Color;
clearBackgroundColor = isClearColor ? camera.backgroundColor.linear : Color.clear;


# [Draw Calls](https://catlikecoding.com/unity/tutorials/custom-srp/draw-calls/)

bool isEnableGpuInstancing;
bool isEnableDynamicBatching;
bool isEnableSrpBathing;
DrawingSettings drawingSettings = new DrawingSettings(unlitShaderTagId, sortingSettings) {
    enableDynamicBatching = isEnableDynamicBatching,
    enableInstancing = isEnableGpuInstancing,
};
GraphicsSettings.useScriptableRenderPipelineBatching = isEnableSrpBathing;



텍스처와 샘플러 상태는 셰이더 리소스입니다.
인스턴스별로 제공할 수 없으며 글로벌 범위에서 선언해야 한다.

TEXTURE2D(_BaseMap);
SAMPLER(sampler_BaseMap);

UNITY_INSTANCING_BUFFER_START(UnityPerMaterial)
	UNITY_DEFINE_INSTANCED_PROP(float4, _BaseMap_ST)
UNITY_INSTANCING_BUFFER_END(UnityPerMaterial)

float4 baseST = UNITY_ACCESS_INSTANCED_PROP(UnityPerMaterial, _BaseMap_ST);



	float2 baseUV : TEXCOORD0;
	float2 baseUV : VAR_BASE_UV; // ??? TODO


# [Directional Lights](https://catlikecoding.com/unity/tutorials/custom-srp/directional-lights/)


		drawingSettings.SetShaderPassName(1, litShaderTagId);

		Light light = RenderSettings.sun;
		light_Color = light.color.linear * light.intensity
		light_Dir   = -light.transform.forward

        commandBuffer.SetGlobalVector

        
        NativeArray<VisibleLight> visibleLights = cullingResults.visibleLights;
        
        ref VisibleLight visibleLight = ref visibleLights[i];
        visibleLight.lightType == LightType.Directional
        light_Color = visibleLight.finalColor;
		light_Dir   = -visibleLight.localToWorldMatrix.GetColumn(2);

        commandBuffer.SetGlobalVectorArray
        
        GraphicsSettings.lightsUseLinearIntensity = true;

BRDF : TODO
[ShaderGUI](https://docs.unity3d.com/ScriptReference/ShaderGUI.html)


# [Directional Shadows](https://catlikecoding.com/unity/tutorials/custom-srp/directional-shadows/)


- 품질
  - 그림자를 얼마나 멀리 렌더링할 것인지
  - 그림자 맵을 얼마나 크게 할 것인지

if (camera.TryGetCullingParameters(out ScriptableCullingParameters p))
{
    float shadowDistance = Mathf.Min(maxShadowDistance, camera.farClipPlane);
    p.shadowDistance = shadowDistance;
}
