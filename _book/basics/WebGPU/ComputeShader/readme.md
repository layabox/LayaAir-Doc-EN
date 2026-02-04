# Compute Shader

> Author: Charley

WebGPU Compute is a general-purpose GPU computing capability provided by WebGPU. It allows developers to directly execute highly parallel, compute-intensive tasks on the GPU through compute shaders in a browser environment, rather than being limited to the traditional graphics pipeline that "only serves rendering." It is a key step in bringing web engines into "modern graphics architecture." Without Compute, WebGPU is just "faster WebGL," but with Compute, it enables web graphics capabilities to gradually possess the modern characteristics of "general-purpose computing."

## 1. Introduction to Compute Shaders

### 1.1 What is a Compute Shader

A Compute Shader is a **general-purpose computing program** that executes on the GPU. It doesn't participate in vertex transformation or fragment shading, nor does it output directly to the screen. Instead, it performs parallel reads and writes to GPU memory through **StorageBuffer, StorageTexture, and other writable resources**. This type of resource read/write is mainly reflected in the WebGPU compute pipeline, where the engine automatically generates binding layouts based on uniformMaps, so developers don't need to manually manage bindings.

Compared to vertex/fragment shaders, the core differences of Compute Shaders are:

- **No fixed rendering stage constraints**: Not dependent on geometry or rasterization
- **Scheduled in workgroups**: Completely developer-controlled parallel granularity
- **Data-oriented rather than graphics-oriented**: Suitable for any "parallelizable, batchable" computing problem

In terms of capability boundaries, it's closer to a lightweight version of CUDA/OpenCL, just running in WebGPU's secure sandbox.

> Only available in WebGPU mode, not supported by WebGL

### 1.2 Core Role of Compute Shaders

The core value of Compute Shaders is not in "being able to compute," but in:

- **Migrating highly parallel, consistently structured computations from CPU to GPU**, reducing CPU pressure
- Compute Shakers can directly read and write Storage Textures and Storage Buffers, **avoiding frequent data round trips and synchronization between CPU ↔ GPU**
- **Enabling GPU to directly participate in the engine runtime system construction process**

Compute Shaders typically handle the following responsibilities:

- GPU culling (Frustum / HiZ / batch-level culling)
- Batching and instance parameter construction (Indirect Draw Args)
- Parallel updates of particle, instance, and animation data
- General-purpose computing for images and volume data (filtering, blur, statistics)

The common characteristic of these tasks is: **large data volume, consistent rules, highly parallelizable**.

### 1.3 Capability Boundaries and Unsuitable Scenarios

Compute Shader is not a "universal function on GPU," it has clear capability boundaries:

- Not suitable for logic with extremely complex branches or severe thread divergence
- Not suitable for algorithms with strong sequential dependencies requiring frequent cross-thread synchronization
- Not suitable for small data volume, low parallelism scattered computations

In these scenarios, CPU is often more efficient and easier to maintain.

Therefore, a healthy engine architecture should follow the principle:

**Compute Shader is for scaled, structured data processing, not business logic itself.**

Only by clarifying this can we avoid the anti-pattern of "using Compute just for the sake of using Compute."

## 2. Engine Support for Compute Shaders

The engine's support for Compute Shaders isn't simply "being able to run a piece of WGSL," but rather establishes a complete set of mechanisms around **engineering usability**, mainly reflected in three levels.

### 2.1 Resource System Integration

The engine has registered the `computeshader` resource loading type, allowing Compute Shaders to exist as resource files and participate in the complete resource lifecycle management (loading, caching, reuse, release).

- Supports `.computeshader` extension
- Internal structure completely reuses `.shader`'s `Shader3D Start / GLSL Start` syntax
- Parsed by `ComputeShaderLoader` and `ComputeShaderParser`

This makes Compute Shaters manageable like materials and Shaders, rather than scattered string code.

### 2.2 Complete GLSL → WGSL Compilation Chain

To reduce learning and migration costs, the engine chooses to **continue using GLSL 450 as the writing language**, and internally completes the conversion to WGSL required by WebGPU:

1. GLSL 450 code concatenation (automatically insert layout/binding)
2. glslang compiles to SPIR-V
3. Naga converts SPIR-V to WGSL

This process is completely automatic, developers don't need to directly touch WGSL or manually maintain binding numbers.

### 2.3 Resource Binding Automation Driven by uniformMaps

All resources of Compute Shaders (uniform, texture, storage buffer, etc.) are declared through `uniformMaps`. The engine, based on these declarations:

- Automatically generates WebGPU bind group layout
- Automatically assigns set/binding indices
- Automatically handles access permissions for StorageBuffer/StorageTexture

Developers only need to ensure **consistency in semantics between GLSL declarations, uniformMaps, and ShaderData**, to complete the full resource binding.

## 3. Overall Compute Shader Development Process

From a macro perspective, the Compute Shader usage process can be understood as a clear pipeline:

**Write compute logic → Declare resources → Create Shader → Prepare parameters → dispatch execution**

### 3.1 Writing GLSL Compute Code

Compute Shaders must declare local workgroup size:

```
layout(local_size_x = 16, local_size_y = 16, local_size_z = 1) in;
```

`local_size` determines the number of threads within a workgroup and is an important parameter for performance tuning.

### 3.2 Declaring uniformMaps

`uniformMaps` is an **array**, where each element corresponds to a bind group. It defines:

- Which resources are used
- Resource types (Uniform/Texture/Storage)
- Access permissions and formats for Storage resources

This is the core contract between Compute Shader and the engine.

### 3.3 Creating ComputeShader

ComputeShader can be created in two ways:

- **Resource file loading**: Suitable for formal projects and maintainable code
- **Dynamic code creation**: Suitable for runtime generation, debugging, experimental logic

Both methods are completely consistent at the underlying level.

### 3.4 Preparing ShaderData

When running a Compute Shader, you must prepare a corresponding `ShaderData` for each uniformMap and **pass them in index order**.

uniformMaps[0] ↔ shaderData[0]
uniformMaps[1] ↔ shaderData[1]

Inconsistent order will cause binding misalignment, which is one of the most common error sources.

### 3.5 Dispatch Execution

Compute Shader execution is recommended to be initiated through ComputeCommandBuffer, first adding dispatch commands, then executing them uniformly at an appropriate time.

The parameters of dispatch represent the number of workgroups, not the total number of threads, so the actual parallel scale needs to be calculated in combination with local_size.

## 4. Resource File Structure and Parsing Process

### 4.1 ComputeShader Loading Entry

`.computeshader` files are loaded by `ComputeShaderLoader`, which internally directly calls `ComputeShaderParser.parse()` to complete parsing and compilation.

From the resource system's perspective, Compute Shader has no essential difference from ordinary Shaders.

### 4.2 Shader3D / GLSL Structure

Compute Shaders use the standard Shader3D structure:

- `type: ComputeShader`
- `uniformMaps`: Resource declarations
- `code`: Points to GLSL code block

Parser is responsible for:

- Type conversion and default value filling for uniformMaps
- Special handling for StorageTexture/StorageBuffer
- GLSL code extraction and preprocessing

## 5. uniformMaps Rules Explained

### 5.1 uniformMaps is an Array, Not an Object

This design directly corresponds to WebGPU's **multi-bind group model**. Each uniformMap is a bind group.

At execution time:

- uniformMaps.length must equal ShaderData[].length
- Index order must be strictly consistent

### 5.2 StorageTexture2D and StorageBuffer Rules

- StorageTexture2D
  - Must declare `format`
  - Must declare `access` (read/write)
- StorageBuffer
  - Must declare `access` (readonly/readwrite)
  - Will map to DeviceBuffer or ReadOnlyDeviceBuffer

Although the engine provides default values, **explicit declaration is strongly recommended** in formal projects.

### 5.3 Auto-Fill Mechanism for Undeclared Resources

When uniform or SSBO exists in GLSL but is not declared in uniformMaps:

The engine will automatically supplement these resources into the first uniformMap during the WebGPU Compute GLSL preprocessing stage and regenerate the bind group layout.

This mechanism is for improving debugging and experimentation efficiency, but is not recommended as a long-term dependency.

## 6. Compilation Process and Underlying Implementation

### 6.1 ComputeShader Creation and Caching

`ComputeShader.createComputeShader()` uses name as key for caching to avoid repeated compilation.

### 6.2 GLSL → SPIR-V → WGSL

When compiling Compute Shaders, `#version 450` is forced, and `std430` is enabled as SSBO layout,

While automatically inserting set/binding declarations.

These are the key foundations for Compute to correctly map to WebGPU.

## 7. Automatic Binding Mechanism and SSBO Constraints

### 7.1 Automatic Binding Point Allocation Principle

Each uniformMap corresponds to a bind group, with bindings within the group incrementing from 0.

Texture will be split into two bindings, Texture and Sampler, in WebGPU's binding layout. Storage resources directly use storage bindings.

Developers don't need to manually maintain binding numbers, but must ensure consistent naming.

### 7.2 SSBO Must Use BlockName Matching

SSBO in GLSL must use:

```typescript
buffer BlockName {
  ...
} instanceName;
```

The engine uses **BlockName** for matching, not the instance name. This means:

- Name declared in uniformMap
- Name assigned in ShaderData
- BlockName in GLSL

All three must be completely consistent.

## 8. Examples

### 8.1 Resource File Method

The following example uses the .shader file structure to define a compute shader resource file.

```glsl
Shader3D Start
{
    type: ComputeShader,
    name: "BlurTexture",
    uniformMaps: [
        {
            inputTex: { type: Texture2D },
            outputTex: { type: StorageTexture2D, format: "rgba8", access: "writeonly" },
        }
    ],
    code: "blur_CS",
}
Shader3D End

GLSL Start

#defineGLSL blur_CS

buffer s_Params {
    int filterDim;
    uint blockDim;
    uint value;
};

shared vec3 tile[4][128];

layout(local_size_x = 32, local_size_y = 1, local_size_z = 1) in;
void main()
{
    int filterOffset = (filterDim - 1) / 2;
    ivec2 dims = textureSize(inputTex, 0);

    ivec2 workGroup = ivec2(int(gl_WorkGroupID.x), int(gl_WorkGroupID.y));
    ivec2 local = ivec2(int(gl_LocalInvocationID.x), int(gl_LocalInvocationID.y));

    ivec2 baseIndex = workGroup * ivec2(int(blockDim), 4) + ivec2(local.x * 4, local.y * 1) - ivec2(filterOffset, 0);

    // load 4x4 block per thread
    for (int r = 0; r < 4; ++r) {
        for (int c = 0; c < 4; ++c) {
            ivec2 loadIndex = baseIndex + ivec2(c, r);
            if (value != 0u) {
                loadIndex = ivec2(loadIndex.y, loadIndex.x);
            }

            vec2 uv = (vec2(loadIndex) + vec2(0.5, 0.5)) / vec2(dims);
            vec3 col = textureLod(inputTex, uv, 0.0).rgb;
            int idx = 4 * local.x + c;
            tile[r][idx] = col;
        }
    }

    // ensure all loads visible
    barrier();

    // compute and write results
    for (int r = 0; r < 4; ++r) {
        for (int c = 0; c < 4; ++c) {
            ivec2 writeIndex = baseIndex + ivec2(c, r);
            if (value != 0u) {
                writeIndex = ivec2(writeIndex.y, writeIndex.x);
            }

            int center = 4 * local.x + c;
            if (center >= filterOffset && center < 128 - filterOffset && all(lessThan(writeIndex, dims))) {
                vec3 acc = vec3(0.0);
                float invFilter = 1.0 / float(filterDim);
                for (int f = 0; f < filterDim; ++f) {
                    int i = center + f - filterOffset;
                    acc += invFilter * tile[r][i];
                }
                imageStore(outputTex, writeIndex, vec4(acc, 1.0));
            }
        }
    }
}
GLSL End
```

This example demonstrates the actual usage of SSBO BlockName (s_Params) and shows the correspondence between StorageTexture2D in uniformMap and GLSL.

### 8.2 Dynamically Creating Compute Shader Using GLSL Code

**GLSL Example Code:**

```glsl
#include "CullResourceCommon.glsl"

layout(local_size_x = 64, local_size_y = 1, local_size_z = 1) in;

void main()
{
    uint instanceIndex = gl_GlobalInvocationID.x;
    uint batchCount = u_getResultParamsArray[0].x;

    if (instanceIndex >= batchCount) {
        return;
    }

    // WGSL: atomicStore(&indirectArgs[instanceIndex].instanceCount, 0u);
    // GLSL: Use atomicExchange for atomic assignment
    atomicExchange(indirectArgs[instanceIndex].instanceCount, 0u);

    indirectArgs[instanceIndex].instanceoffset = batchPosBuffer[instanceIndex];
}
```

**TypeScript Creation and Binding Example Code:**

```typescript
let uniformMap = Laya.LayaGL.renderDeviceFactory.createGlobalUniformMap("ComputeShaderName");
uniformMap.addShaderUniform(Laya.Shader3D.propertyNameToID("u_Image", "u_Image", Laya.ShaderDataType.StorageTexture2D, {format: "rgba8", access: "writeonly"});

let computeShader = Laya.ComputeShader.createComputeShader(`ComputeShaderName`, ComputeCode, [uniformMap]);
```

**Key Points:**

- `IndirectArgs` is the SSBO BlockName
- Names in uniformMap, ShaderData, and GLSL must be consistent
- Access permissions of StorageBuffer must match GLSL behavior

### 8.3 Correspondence Between ShaderData and Dispatch

When executing Compute Shaders, it's recommended to use ComputeCommandBuffer to add dispatch commands and execute them at a unified timing.

This avoids directly calling the underlying compute context. The array order of ShaderData must correspond to uniformMaps.

Example as follows:

```typescript
const cmd = new Laya.ComputeCommandBuffer();
cmd.addDispatchCommand(Laya.computeShader, Laya.shaderDefine, [shaderData0, shaderData1], new Laya.Vector3(x, y, z));
cmd.executeCMDs();
```

## 9. Common Questions and Best Practices

Problems in the early learning stage of Compute Shaders are almost all concentrated in **inconsistent declarations**:

- uniformMaps and ShaderData order inconsistent
- SSBO BlockName inconsistent
- StorageTexture not declaring format/access

Recommendations:

- Make all resource declarations explicit and complete
- Maintain strict naming consistency
- Minimize reliance on auto-fill mechanisms

Once Compute Shaders can run stably, gradually leverage these mechanisms to improve development efficiency.
