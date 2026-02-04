# Presets

> Author: Charley

Presets are used to define default behaviors and editing environment for the project in the IDE. They don't directly affect the final build output, but determine **default properties for new resources, editing work environment, and scene and object organization**. Properly configuring presets can reduce repetitive setup costs and maintain consistent editing experience in team collaboration.

These presets mainly affect:

- Default types for new resources
- Prefab and scene editing environment
- 2D / 3D object layer hierarchy

## 1. Texture Type (`textureType`)

`textureType` is used to set the **default processing type for newly created texture resources**. It determines which preset rules the IDE uses for parsing and management when importing or creating texture resources.

Without explicitly modifying individual resource properties, this value takes effect as the default, so it's more suitable for matching the overall resource type of the project rather than solving individual resource differences.

For most projects, using the default type works well. However, in projects with highly concentrated resource types, setting an appropriate default value in advance can significantly reduce subsequent manual adjustment work.

**Usage Recommendations**

- For projects mainly using 2D sprites, set it to Sprite Texture to make new textures naturally conform to 2D usage habits
- For projects mainly using 3D lightmaps, set it to Lightmap to reduce duplicate configuration of lighting-related resources
- For projects with mixed resource types, recommend keeping the **default value** and distinguishing through individual resource properties

Note that this configuration **only affects newly created resources**, not batch modification of existing textures.

## 2. Prefab Editing Environment (`prefabEditEnv`)

`prefabEditEnv` is used to specify the **scene environment used by prefabs in edit mode**. When opening a prefab for editing, the IDE doesn't directly use the game runtime scene, but loads the environment scene specified by this configuration as the "editing background."

The IDE includes a built-in default prefab editing environment with basic camera and lighting settings that can meet most prefab editing needs. However, in 3D projects or scenarios with higher visual effect requirements, using a custom editing environment can significantly improve preview accuracy.

**Applicable Scenarios**

- 3D Prefabs: Provide lighting and camera closer to actual runtime effects through custom environments
- Effect or Material Preview: Verify effect performance in specific environments
- Team Collaboration: Unify prefab editing environment to avoid different members seeing inconsistent effects

## 3. Auto Bake (`autoBake`)

`autoBake` is used to control whether to **automatically trigger light baking** when lighting parameters change during 3D scene editing. This option directly affects real-time feedback efficiency and performance consumption during the editing phase.

When scene structure is relatively simple or in the lighting debugging phase, enabling auto-bake can quickly see lighting change results.

For example: In Scene3D, when changing skybox material (Material changed from skybox to other material), Reflection Probe's IBL Texture doesn't need manual clicking of the `Bake` button. After saving the scene, the IDE will automatically re-bake.

> Only when Reflection Probe's Source is Skybox will it auto-bake. Custom cannot auto-bake.

However, in complex scenes, frequent auto-baking may significantly reduce editing smoothness.

**Usage Trade-off**

- Lighting frequent adjustment phase: Enable to improve feedback efficiency
- Large scene scale or performance constrained: Disable, change to manual bake timing control

Note: This configuration only affects editing behavior, not runtime lighting results.

## 4. 3D Layer Names Definition (`layers`)

`layers` is used to define available layer names in 3D scenes, an important foundation for 3D object grouping management. Layers not only affect editing organization but are also commonly used for camera culling and rendering control division.

The system includes a built-in layer named `Default` at index 0, which cannot be modified or deleted.

**Typical Uses**

- Distinguish objects by function (character / environment / effects / props)
- Work with camera layer masks to control rendering range

Layer quantity and naming should be planned as early as possible in the project to reduce later adjustment costs.

> For more introduction to Layer, refer to the documentation ["Using 3D Sprites"](../../../../3D/Sprite3D/readme.md).

## 5. 2D Layer Names Definition (`2DLayers`)

`2DLayers` is used to define available layer names in 2D scenes, positioned similarly to 3D's `layers`, but more commonly used in render order and interaction management.

Consistent with 3D layers, the `Default` layer at index 0 is a system-reserved layer and cannot be modified. Also, the total number of 2D layers has an upper limit, maximum 32.

**Usage**

In 2D node rendering-related components (such as: 2D Mesh Renderer, 2D Trail Renderer, 2D Line Renderer, 2D Lights, etc.), you can set render layers, commonly used for lights and shadows to specify affected layers.
