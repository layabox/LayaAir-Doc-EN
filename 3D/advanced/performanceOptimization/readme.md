# 3D performance optimization

## 1. Basic understanding of graphics performance

### 1.1 Reduce the CPU cost of rendering

During the rendering process, the factor that has the greatest impact on CPU rendering time is the cost of sending rendering instructions to the GPU. Rendering instructions include Draw Calls and commands that change settings on the GPU before drawing 3D objects.

The following methods can reduce the CPU cost of rendering:

- Reduce the number of rendered objects in the scene
  - Consider reducing the total number of objects in the scene, e.g. using skyboxes instead of rendering distant 3D objects
  - Perform more efficient culling of objects in the scene to reduce rendering pressure on the engine.
- Reduce the number of times the object is rendered
  - Where appropriate, use LightMap to bake lights and shadows. This operation will increase video memory usage and build time, but can improve running efficiency.
  - Reduce the number of light sources
  - Use real-time shadows with caution
  - Use reflection probes with caution

### **1.2 Reduce GPU cost of rendering**

Limited by memory bandwidth, the texture size is too high and the number of textures is too large, which will cause a GPU rendering bottleneck.

- Enable mipmaps for textures whose distance from the camera changes at runtime. (For example, most textures used in 3D scenes). This increases the memory usage and storage space of these textures, but improves runtime GPU performance.

- Use a suitable compression format to reduce the size of textures in memory. This reduces load times, reduces memory footprint, and improves GPU rendering performance. Compressed textures use only a fraction of the memory bandwidth required by uncompressed textures.

If the application is vertex processing bound, it means that the GPU can process more vertices during the vertex processing stage

- Reduce vertex shader execution cost.

- Optimize geometry: don't use unnecessary triangles and try to keep the number of UV mapped seams and hard edges (double vertices) as low as possible.
- Use LOD to optimize different Mesh types and optimize the number of vertices.

## 2. Optimize Draw Call

Because rendering state changes can be resource-intensive, it's important to optimize them. The main way to optimize render state changes is to reduce their number. There are two ways to do this:

- Reduce the total number of draw calls. When you reduce the number of draw calls, you also reduce the number of render state changes between them.
- Organize draw calls in a way that reduces the number of changes to rendering state. If the graphics API can perform multiple draw calls using the same rendering state, the draw calls can be grouped together without performing as many render state changes.

The following methods are provided in LayaAir:

- GPU instance
- Dynamic Batch
- Static Batch
- Custom Static Batch

## 3.GPU instance

GPU instancing is a draw call optimization method that renders multiple copies of a mesh with the same material in a single draw call. Each copy of the grid is called an instance. This is useful for drawing things that appear multiple times in a scene, such as trees or grass.

GPU instances render the same mesh in the same draw call. To add variety and reduce a repetitive look, each instance can have a different property, such as Color or Scale. Draw calls that render multiple instances appear in the framework debugger draw grid(instances).

GPU Instance requires hardware support. Make sure the hardware you are currently using can support GPU Instance rendering.

 ![image-20230217113033073](img/image-20230217113033073.png)

The above picture shows a GPU Instance test scene and the detailed drawing information corresponding to the test scene. In the picture, red-green-blue-yellow are four different materials.

 There are currently only three Instance DrawCalls, and the engine automatically executes the Instance rendering process for objects that meet the Instance conditions.

Engine Instance rendering conditions:

- Same Mesh
- same material
- enableInstance (customize the switch on the Shader, the engine's default shader turns on Instance)
- Whether the shadow status is the same (whether to receive shadow)
- Whether the reflection probe status is the same

Red-green-blue objects have different Mesh and different materials, but every red, green or blue object has the same material, the same Mesh, the same Instance state, the same shadow state, Same reflection probe status. Therefore, these three types of objects comply with the engine Instance rendering judgment process. The engine automatically instantiates these three types of objects and instantiates all objects of each color into one InstanceDraw Call to complete the rendering process. Yellow objects meet almost all the requirements for Instance rendering judgment. However, because of different Mesh grid data, the engine will not perform Instance rendering on all yellow objects.

In summary, a basic Instance rendering condition can be summarized into three points: enableInstance is turned on, the same Mesh, and the same material.

If you need to customize your own personalized Instance rendering judgment, developers need to organize the rendered data themselves in the form of [CommandBuffer](../CommandBuffer/readme.md)

## 4.Dynamic Batch

Dynamic merging is divided into two types: **instance merging** and **vertex merging**. Both optimizations require no settings from the developer, and objects can move dynamically without restrictions. However, the merger principle is relatively strict. The following are the most basic conditions for the two mergers.

**Instance merge:**

 Both conditions of the same Mesh and the same material need to be met. In a three-dimensional scene, there may still be a large number of models with the same material as Mesh, and there is a lot of room for instance merging at this time.

**Vertex merge:**

 The same material is required and the model vertices are less than 10. Vertex merging currently has room for use on some fake shadow and special effects models.

**Note:** Translucent objects require continuous rendering to be dynamically merged, so the probability of dynamic merging of translucent objects is low.

**Turn off dynamic batching option**

In the engine's Config3D.ts file, enableDynamicBatch value option, true means turning on dynamic batching, false means turning off dynamic batching.

 ![image-20221226101159327](img/image-20221226101159327.png)

Pic 4-1

## 5.Static Batch

Static batching is a draw call batching method that combines non-moving meshes to reduce draw calls. It converts the combined meshes to world space and builds a shared vertex and index buffer for them. Then, for the visible mesh, the engine performs a series of simple draw calls with almost no state changes between each call. Static batching does not reduce the number of draw calls, but rather the number of render state changes between them. Static batching is more efficient than dynamic batching because static batching does not transform vertices on the CPU.

**Turn off static batching option**

In the engine's Config3D.ts file, enableStaticBatch value option, true means turning on dynamic batching, false means turning off dynamic batching.

 ![image-20221226103109634](img/image-20221226103109634.png)

Figure 5-1

Conditions for static batching:

- The object is Static (including sub-objects)
- Unified model using the same material

 ![image-20221226103613220](img/image-20221226103613220.png)

Figure 5-2

## 6.Custom Static Batch(Static Batch Volume component)

In the Object's inspect panel, add a component, select the Rendering option, and find the Static Batch Volume component.

 ![image-20221226103902624](img/image-20221226103902624.png)

Figure 6-1

Drag the small white dot in the Scene window to select the appropriate Volume size.

 ![image-20221226104054399](img/image-20221226104054399.png)

Figure 6-2

Use of the Static Batch Volume component: After the Volume box above is selected to the appropriate size, in the component's details panel, check Static Instance Batch, and then click reBatch. The selected object in the Volume will perform the Batch operation, optimizing Draw. Call to improve operational efficiency. The Batch component with the CheckLOD option checked will automatically check the object LOD attribute information in the Volume, and then divide all objects in the Volume into different LOD rendering objects according to the object LOD level of the LOD Cull Rate Array.

 ![image-20221226104233853](img/image-20221226104233853.png)

Figure 6-3



 ![image-20230117103118514](img/image-20230117103118514.png)

Figure 6-4
## 7\. Node-Based Material Batching Feature

### 7.1 Visual Effect

The node-based material batching feature can be demonstrated through Figure 7-1 and Figure 7-2. This test scene contains a total of 200 small spheres and 1 plane. After batching, the final scene's **opaque draw calls** are significantly reduced.

![1](img/1.png)

Figure 7-1: Scene Screenshot

![2](img/2.png)

Figure 7-2: Stat Panel

### 7.2 Usage Example

This example uses a script to implement a node-based material batching feature. It primarily optimizes rendering performance by assigning shared materials and meshes to multiple 3D nodes and utilizing a custom `UniformBuffer`.

The script code is as follows:

```typescript
const { regClass, property } = Laya;

@regClass()
export class Script extends Laya.Script {
    // Shared material for batching
    public batchMat: Laya.Material;
    // Number of colors
    private _colorNums = 20;
    // Number of sprites
    private _spriteNums = 200;

    private _createColorBufferData() {
        // Randomly generate 20 color values.
        let colorBuffer = new Float32Array(20 * 4);
        for (var i = 0; i < this._colorNums; i++) {
            let offset = i * 4;
            colorBuffer[offset] = Math.random();
            colorBuffer[offset + 1] = Math.random();
            colorBuffer[offset + 2] = Math.random();
            colorBuffer[offset + 3] = 1;
        }
        // Set the uniform buffer.
        this.batchMat.setBuffer("colormap", colorBuffer);
    }

    // Randomly generate _spriteNums number of spheres with random colors.
    private _createMeshSpriteRender() {
        let mesh = Laya.PrimitiveMesh.createSphere(0.5);
        let ownerSprite = this.owner;
        let positionRanvge = 30;
        for (var i = 0; i < this._spriteNums; i++) {
            let sprite = ownerSprite.addChild(new Laya.Sprite3D());
            let filter = sprite.addComponent(Laya.MeshFilter);
            let render = sprite.addComponent(Laya.MeshRenderer);
            // Set the same material and mesh.
            filter.sharedMesh = mesh;
            render.sharedMaterial = this.batchMat;
            // Set a random position.
            sprite.transform.localPosition = this._getRandomPosition(positionRanvge);
            // Get a random color index.
            let colorIndex = Math.floor(Math.random() * this._colorNums);
            // Set the node's Laya.ENodeCustomData.custom_0 to the corresponding color index.
            render.setNodeCustomData(Laya.ENodeCustomData.custom_0, colorIndex);
        }
    }
    
    private _getRandomPosition(positionRanvge: number): Laya.Vector3 {
        let getRangeRandom = () => {
            return (Math.random() - 0.5) * positionRanvge;
        }
        return new Laya.Vector3(getRangeRandom(), 0.3, getRangeRandom());
    }

}
```

The shader code is as follows:

```glsl
Shader3D Start
{
    type:Shader3D
    name:PBRColorBatchShader
    enableInstancing:true,
    supportReflectionProbe:true,
    uniformMap:{
        u_AlphaTestValue: { type: Float, default: 0.5, range: [0.0, 1.0] },

        u_TilingOffset: { type: Vector4, default: [1, 1, 0, 0] },

        u_AlbedoColor: { type: Color, default: [1, 1, 1, 1] },
        u_AlbedoTexture: { type: Texture2D, options: { define: "ALBEDOTEXTURE" } },

        u_NormalTexture: { type: Texture2D, options: { define: "NORMALTEXTURE" } },
        u_NormalScale: { type: Float, default: 1.0, range: [0.0, 2.0] },

        u_Metallic: { type: Float, default: 0.0, range: [0.0, 1.0] },
        u_Smoothness: { type: Float, default: 0.0, range: [0.0, 1.0] },
        u_MetallicGlossTexture: { type: Texture2D, options: { define: "METALLICGLOSSTEXTURE" } },

        u_OcclusionTexture: { type: Texture2D, options: { define: "OCCLUSIONTEXTURE" } },
        u_OcclusionStrength: { type: Float, default: 1.0 },

        u_EmissionColor: { type: Color, default: [0, 0, 0, 0] },
        u_EmissionIntensity: { type: Float, default: 1.0 },
        u_EmissionTexture: { type: Texture2D, options: { define: "EMISSIONTEXTURE" } },
    },
    defines: {
        EMISSION: { type: bool, default: false },
        ENABLEVERTEXCOLOR: { type: bool, default: false }
    }
    shaderPass:[
        {
            pipeline:Forward,
            VS:LitVS,
            FS:LitFS
        }
    ]
}
Shader3D End

GLSL Start
#defineGLSL LitVS
    #define SHADER_NAME PBRColorBatchShader

    #include "Math.glsl";

    #include "Scene.glsl";
    #include "SceneFogInput.glsl"

    #include "Camera.glsl";
    #include "Sprite3DVertex.glsl";

    #include "VertexCommon.glsl";

    #include "PBRVertex.glsl";

    varying float spriteCustomData;

    void main()
    {
        Vertex vertex;
        getVertexParams(vertex);

        PixelParams pixel;
        initPixelParams(pixel, vertex);

        gl_Position = getPositionCS(pixel.positionWS);

        gl_Position = remapPositionZ(gl_Position);
        
        spriteCustomData = NodeCustomData0;

    #ifdef FOG
        FogHandle(gl_Position.z);
    #endif // FOG
    }
#endGLSL

#defineGLSL LitFS
    #define SHADER_NAME PBRColorBatchShader

    #include "Color.glsl";

    #include "Scene.glsl";
    #include "SceneFog.glsl";

    #include "Camera.glsl";
    #include "Sprite3DFrag.glsl";

    #include "PBRMetallicFrag.glsl";

    uniform vec4 colormap[20];
    varying float spriteCustomData;

    void initSurfaceInputs(inout SurfaceInputs inputs, inout PixelParams pixel)
    {
        inputs.alphaTest = u_AlphaTestValue;

    #ifdef UV
        vec2 uv = transformUV(pixel.uv0, u_TilingOffset);
    #else // UV
        vec2 uv = vec2(0.0);
    #endif // UV

        inputs.diffuseColor = colormap[int(spriteCustomData)].rgb;
        inputs.alpha = colormap[int(spriteCustomData)].a;

    #ifdef COLOR
        #ifdef ENABLEVERTEXCOLOR
        inputs.diffuseColor *= pixel.vertexColor.xyz;
        inputs.alpha *= pixel.vertexColor.a;
        #endif // ENABLEVERTEXCOLOR
    #endif // COLOR

    #ifdef ALBEDOTEXTURE
        vec4 albedoSampler = texture2D(u_AlbedoTexture, uv);
        #ifdef Gamma_u_AlbedoTexture
        albedoSampler = gammaToLinear(albedoSampler);
        #endif // Gamma_u_AlbedoTexture
        inputs.diffuseColor *= albedoSampler.rgb;
        inputs.alpha *= albedoSampler.a;
    #endif // ALBEDOTEXTURE

        inputs.normalTS = vec3(0.0, 0.0, 1.0);
    #ifdef NORMALTEXTURE
        vec3 normalSampler = texture2D(u_NormalTexture, uv).rgb;
        normalSampler = normalize(normalSampler * 2.0 - 1.0);
        normalSampler.y *= -1.0;
        inputs.normalTS = normalScale(normalSampler, u_NormalScale);
    #endif

        inputs.metallic = u_Metallic;
        inputs.smoothness = u_Smoothness;

    #ifdef METALLICGLOSSTEXTURE
        vec4 metallicSampler = texture2D(u_MetallicGlossTexture, uv);
        inputs.metallic = metallicSampler.x;
        inputs.smoothness = (metallicSampler.a * u_Smoothness);
    #endif // METALLICGLOSSTEXTURE

        inputs.occlusion = 1.0;
    #ifdef OCCLUSIONTEXTURE
        vec4 occlusionSampler = texture2D(u_OcclusionTexture, uv);
        float occlusion = occlusionSampler.g;
        inputs.occlusion = (1.0 - u_OcclusionStrength) + occlusion * u_OcclusionStrength;
    #endif // OCCLUSIONTEXTURE

        inputs.emissionColor = vec3(0.0);
    #ifdef EMISSION
        inputs.emissionColor = u_EmissionColor.rgb * u_EmissionIntensity;
        #ifdef EMISSIONTEXTURE
        vec4 emissionSampler = texture2D(u_EmissionTexture, uv);
        #ifdef Gamma_u_EmissionTexture
        emissionSampler = gammaToLinear(emissionSampler);
        #endif // Gamma_u_EmissionTexture
        inputs.emissionColor *= emissionSampler.rgb;
        #endif // EMISSIONTEXTURE
    #endif // EMISSION
    }

    void main()
    {
        PixelParams pixel;
        getPixelParams(pixel);

        SurfaceInputs inputs;
        initSurfaceInputs(inputs, pixel);

        vec4 surfaceColor = PBR_Metallic_Flow(inputs, pixel);
        
    #ifdef FOG
        surfaceColor.rgb = sceneLitFog(surfaceColor.rgb);
    #endif // FOG

        gl_FragColor = surfaceColor;

        gl_FragColor = outputTransform(gl_FragColor);
    }
#endGLSL

GLSL End
```

#### 7.2.1 Principle Introduction

Developers can set a corresponding **uniform buffer** using a material's `setBuffer` method.

In the example, we create a `Float32Array` of length 4 \* 20 (i.e., `colorBuffer`) to serve as the data source for `uniform vec4 colormap[20]`, containing 20 different color values.

During implementation, we generate a random color index (`colorIndex`) for each node and store these index values in the node's custom data area at the `ENodeCustomData.custom_0` location using the `BaseRender.setNodeCustomData` method. It's important to note that this interface only supports setting numerical data.

During the rendering phase, the engine detects 200 spheres using the same material and mesh and automatically performs batching optimization. The `CustomData` from different nodes is submitted together in the form of an `InstanceBuffer`, so all 200 spheres can be rendered in just **one draw call**, which greatly improves rendering efficiency.

#### 7.2.2 Important Notes

1.  Currently, the `BaseRender.setNodeCustomData` method only has `custom_0`, `custom_1`, and `custom_2` options.
2.  When a material switches to Instance rendering, these three slots occupy the vertices `VertexMesh.MESH_CUSTOME0`, `VertexMesh.MESH_CUSTOME1`, and `VertexMesh.MESH_CUSTOME2`.
3.  When using a uniform buffer, be aware that excessively large data on low-end mobile devices may run the risk of exceeding the uniform size limit.



