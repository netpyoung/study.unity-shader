

[SerializeField]
ComputeShader computeShader;


``` hlsl
#pragma kernel CSMain

StructuredBuffer<int>   inputData;  // readonly
RWStructuredBuffer<int> outputData; // read/write

[numthreads(20,1,1)]
void CSMain (uint3 id : SV_DispatchThreadID)
{
    outputData[id.x] = 2 * inputData[id.x];
}
```



``` cs
GraphicsBuffer _inputBuffer;
GraphicsBuffer _outputBuffer;

public ComputePass()
{
    List<int> list = Enumerable.Range(start: 0, count: 20).ToList();

    _inputBuffer = new GraphicsBuffer(GraphicsBuffer.Target.Structured, 20, sizeof(int));
    _inputBuffer.SetData(list);
    _outputBuffer = new GraphicsBuffer(GraphicsBuffer.Target.Structured, 20, sizeof(int));
    _outputBuffer.SetData(list);
}



class PassData
{
    public ComputeShader cs;
    public BufferHandle input;
    public BufferHandle output;
}



ComputeShader _cs;
int[] _outputData = new int[20];

public override void RecordRenderGraph(RenderGraph renderGraph, ContextContainer frameData)
{
    _outputBuffer.GetData(_outputData);
    Debug.Log($"Output from compute shader: {string.Join(", ", _outputData)}");

    BufferHandle inputHandle = renderGraph.ImportBuffer(_inputBuffer);
    BufferHandle outputHandle = renderGraph.ImportBuffer(_outputBuffer);

    using (IComputeRenderGraphBuilder builder = renderGraph.AddComputePass("ComputePass", out PassData passData))
    {
        passData.cs = _cs;
        passData.input = inputHandle;
        passData.output = outputHandle;

        builder.UseBuffer(passData.input);
        builder.UseBuffer(passData.output, flags: AccessFlags.Write);
        builder.SetRenderFunc((PassData data, ComputeGraphContext cgContext) => ExecutePass(data, cgContext));
    }
}

static void ExecutePass(PassData data, ComputeGraphContext cgContext)
{
    cgContext.cmd.SetComputeBufferParam(data.cs, data.cs.FindKernel("CSMain"), "inputData", data.input);
    cgContext.cmd.SetComputeBufferParam(data.cs, data.cs.FindKernel("CSMain"), "outputData", data.output);
    cgContext.cmd.DispatchCompute(data.cs, data.cs.FindKernel("CSMain"), threadGroupsX: 1, threadGroupsY: 1, threadGroupsZ: 1);
}
```