# Dynamic Atlas

## I. Overview

An Atlas is a resource optimization technique that combines multiple small textures into a single large texture. It can effectively reduce DrawCalls, decrease texture switching times, and minimize I/O requests. It is one of the key methods for improving rendering performance and memory utilization in 2D products.

Prior to LayaAir 3.4.0, the engine required developers to either pre-create atlas files or configure automatic atlas rules to generate atlas textures and data uniformly during preview or publishing stages. While this approach works well in projects with stable resource structures, it can lead to issues such as atlas redundancy, wasted memory, and impacted loading efficiency in projects with large numbers of scattered textures or inconsistent resource loading sets across different users.

The DynamicAtlasManager feature effectively solves this problem. Dynamic atlases can dynamically merge textures at runtime based on actual loaded content, significantly improving rendering performance and resource utilization while maintaining flexibility.



## II. Using Dynamic Atlas

### 2.1 Creating the Manager

To use dynamic atlases, you need to instantiate a `DynamicAtlasManager` object in your script:

```typescript
//Interface
constructor(config?: Partial<DynamicAtlasConfig>, autoReplace: boolean = true)
```

Parameter Description:

`config` is the configuration parameter for creating the dynamic atlas. The details of each parameter are as follows:

| Parameter         | Type                | Default     | Description                                                    |
| ----------------- | ------------------- | ----------- | -------------------------------------------------------------- |
| largeTextureSize  | [number, number]    | [1024, 1024] | Large texture size [width, height]                             |
| maxLargeTextures  | number              | 4           | Maximum number of large textures                               |
| textureUnitSize   | number              | 16          | Small texture unit size (for space division)                   |
| extendSize        | number              | 1           | Texture extension size (to prevent texture sampling bleeding)  |
| textureFormat     | RenderTargetFormat  | R8G8B8A8    | Texture format                                                 |
| immediately       | boolean             | false       | Whether to execute merge immediately (true=synchronous, false=delayed to next frame) |
| autoExtend        | boolean             | true        | Whether to automatically expand large texture count when space is insufficient |
| checkDuplicate    | boolean             | true        | Whether to check for duplicate textures (to avoid adding duplicates) |

`autoReplace`: Whether to automatically replace original textures. When set to true, after adding textures, the original texture objects will be automatically replaced with texture references from the atlas, without the need to manually call replacement methods.

```typescript
//Usage example
const config: Partial<DynamicAtlasConfig> = {
    largeTextureSize: [512, 512],  // Large texture size 512x512
    maxLargeTextures: 2,           // Maximum 2 large textures
    textureUnitSize: 16,           // Small texture unit size 16x16
    extendSize: 1,                 // Texture extension size 1 pixel
    immediately: true,             // Execute merge immediately
    autoExtend: true,              // Automatically expand large texture count
    checkDuplicate: true           // Check for duplicates
};
this.dynamicAtlasManager = new DynamicAtlasManager(config, true);
```



### 2.2 Adding Textures to the Atlas

#### Adding a Single Texture Object

```typescript
//Interface
addTexture(texture: Texture, scale: number = 1.0, largeTextureIndex: number = -1): boolean
```

Parameter Description:

`texture`: The Texture object to add.

`scale`: Scaling factor.

`largeTextureIndex`: The index of the large texture, specifying which large texture to add to.

```typescript
//Usage example
const texture = Laya.loader.getRes("resources/hero.png", Laya.Loader.IMAGE);
const success = atlasManager.addTexture(texture, 0.5, 0); // Add to specified atlas, scaled to 0.5
```

---

#### Adding Textures via URL

```typescript
//Interface
addTextureByUrl(url: string, scale: number = 1.0, largeTextureIndex: number = -1): boolean
```

When adding textures this way, ensure the texture has been preloaded via Laya.loader.load().

Parameter Description:

`url`: The resource URL.

`scale`: Scaling factor.

`largeTextureIndex`: The index of the large texture, specifying which large texture to add to.

```typescript
//Usage example
Laya.loader.load("resources/hero.png").then(() => {
    // Add to atlas after loading is complete
    atlasManager.addTextureByUrl("resources/hero.png");
});
```

---

#### Batch Adding Textures

```typescript
//Interface
addTextures(textures: Texture[], scale: number = 1.0, largeTextureIndex: number = -1): number
addTexturesByUrl(urls: string[], scale: number = 1.0, largeTextureIndex: number = -1): number
```

Batch adding multiple textures improves efficiency. The interface returns the number of textures successfully added.

Parameter Description:

`textures`: Array of textures to add.

`urls`: Array of resource URLs to add.

`scale`: Scaling factor.

`largeTextureIndex`: The index of the large texture, specifying which large texture to add to.

```typescript
//Usage example
const textures = [texture1, texture2, texture3];
const successCount = atlasManager.addTextures(textures);
console.log(`Successfully added ${successCount} textures`);

const urls = [
    "resources/ui/button.png",
    "resources/ui/panel.png",
    "resources/ui/icon.png"
];
Laya.loader.load(urls).then(() => {
    atlasManager.addTexturesByUrl(urls);
});
```



### 2.3 Removing Textures

#### Removing Specified Texture

```typescript
//Interface
removeTexture(textureId: number, largeTextureIndex: number = -1, event: boolean = true): boolean
```

Parameter Description:

`textureId`: The ID of the texture (can be obtained from texture.bitmap.id)

`largeTextureIndex`: Specifies which large texture to remove from (-1 means remove from all large textures)

`event`: Whether to trigger Event.CHANGE event (default true)

```typescript
//Usage example
const texture = Laya.loader.getRes("resources/hero.png", Laya.Loader.IMAGE);
const textureId = texture.bitmap.id;
atlasManager.removeTexture(textureId);
```

---

#### Removing Textures via URL

```typescript
//Interface
removeTextureByUrl(url: string, largeTextureIndex: number = -1): boolean
```

Parameter Description:

`url`: Texture URL.

`largeTextureIndex`: Specifies which large texture to remove from (-1 means remove from all large textures).

```typescript
//Usage example
atlasManager.removeTextureByUrl("resources/hero.png");
```



### 2.4 Manually Replacing Textures

#### Replacing a Single Texture

```typescript
//Interface
replaceOriginalTexture(textureId: number): boolean
```

Manually replaces original texture objects with texture references from the atlas. When creating the manager with autoReplace=false, you need to manually control the replacement timing.

Parameter Description:

`textureId`: The texture index.

```typescript
//Usage example
// Create a manager without auto-replacement
const atlasManager = new DynamicAtlasManager({}, false);

// Add texture
atlasManager.addTexture(texture);

// Manually trigger replacement (e.g., replace uniformly at end of frame)
atlasManager.replaceOriginalTexture(texture.bitmap.id);
```

---

#### Batch Replacing Textures

```typescript
//Interface
replaceOriginalTextures(textureIds?: number[]): number
```

Batch manually replaces original texture objects. If no parameter is passed or an empty array is passed, all textures that have been added to the atlas are replaced.

Parameter Description:

`textureIds`: Array of texture IDs to replace. If empty or not passed, replaces all added textures.

```typescript
//Usage example
const ids = [texture1.bitmap.id, texture2.bitmap.id];
const count = atlasManager.replaceOriginalTextures(ids);
console.log(`Successfully replaced ${count} textures`);
```



### 2.5 Querying Texture Information

#### Getting Texture Information Object

```typescript
//Interface
getTextureInfo(textureId: number): TextureInfo | null
```

Gets detailed information about a specific texture in the atlas.

Parameter Description:

`textureId`: The texture ID.

Return value: TextureInfo object or null (if it doesn't exist).

Information contained in TextureInfo:

- `source`: Original Texture2D object
- `textureId`: Texture ID
- `url`: Texture URL
- `uv`: UV coordinates in the large atlas (Vector4)
- `largeTextureIndex`: Index of the large texture it belongs to
- `isInAtlas`: Whether it has been added to the atlas
- `merged`: Whether merge is complete
- `referenceCount`: Reference count

```typescript
//Usage example
const texture = Laya.loader.getRes("resources/hero.png", Laya.Loader.IMAGE);
const info = atlasManager.getTextureInfo(texture.bitmap.id);

if (info) {
    console.log(`Texture located in atlas ${info.largeTextureIndex}`);
    console.log(`UV coordinates: ${info.uv}`);
    console.log(`Is merged: ${info.merged}`);
}
```

---

#### Getting Texture Information via URL

```typescript
//Interface
getTextureInfoByUrl(url: string): TextureInfo | null
```

Gets detailed information about a texture in the atlas via resource URL.

Parameter Description:

`url`: The texture resource URL.

Return value: TextureInfo object or null (if it doesn't exist).

```typescript
//Usage example
const info = atlasManager.getTextureInfoByUrl("resources/hero.png");

if (info) {
    console.log(`Texture URL: ${info.url}`);
    console.log(`Large texture index: ${info.largeTextureIndex}`);
}
```

---

#### Getting Large Texture Object

```typescript
//Interface
getLargeTexture(largeTextureIndex: number): LargeTexBase
```

Gets the underlying large texture object at the specified index, for debugging or advanced operations.

Parameter Description:

`largeTextureIndex`: Index of the large texture.

Return value: LargeTexBase object.

```typescript
//Usage example
// Get first large texture
const largeTex = atlasManager.getLargeTexture(0);
console.log(`Large texture size: ${largeTex.width} x ${largeTex.height}`);
```

---

#### Getting All Large Texture Objects

```typescript
//Interface
getAllLargeTextures(): LargeTexBase[]
```

Gets an array of all underlying large texture objects, for debugging or advanced operations.

Return value: Array of LargeTexBase objects.

```typescript
//Usage example
const allTextures = atlasManager.getAllLargeTextures();
console.log(`Currently using ${allTextures.length} large textures`);

// Iterate through all large textures
allTextures.forEach((largeTex, index) => {
    console.log(`Large texture ${index}: ${largeTex.width} x ${largeTex.height}`);
});
```

---

#### Getting Texture Count

```typescript
//Interface
getTextureCount(): number
```

Gets the total number of textures currently in the atlas.

Return value: Number of textures.

```typescript
//Usage example
const count = atlasManager.getTextureCount();
console.log(`Total of ${count} textures in atlas`);
```



### 2.6 Statistics and Monitoring

#### Getting Statistics

```typescript
//Interface
getStatistics(): object
```

Gets detailed statistics about the atlas, for performance monitoring and debugging.

Return value: Statistics object containing the following fields:

- `textureCount`: Number of textures in the atlas
- `usageRate`: Atlas space usage rate (0-1)
- `gpuMemoryUsage`: GPU memory usage (bytes)
- `largeTextureCount`: Current number of large textures
- `config`: Configuration information

```typescript
//Usage example
const stats = atlasManager.getStatistics();

console.log(`Texture count: ${stats.textureCount}`);
console.log(`Space usage rate: ${(stats.usageRate * 100).toFixed(2)}%`);
console.log(`Memory usage: ${(stats.gpuMemoryUsage / 1024 / 1024).toFixed(2)} MB`);
console.log(`Large texture count: ${stats.largeTextureCount}/${stats.config.maxLargeTextures}`);

// Periodic monitoring
Laya.timer.loop(1000, this, () => {
    const stats = atlasManager.getStatistics();
    if (stats.usageRate > 0.9) {
        console.warn("Atlas space usage exceeds 90%, consider increasing maxLargeTextures");
    }
});
```



### 2.7 Cleanup and Destruction

#### Cleaning Up Unused Textures

```typescript
//Interface
cleanupUnusedTextures(forceClean: boolean = false): number
```

Cleans up textures with reference count of 0, releasing atlas space. Suitable for cleaning old scene textures after scene switching, periodic cleanup to release memory, or resource reorganization.

Parameter Description:

`forceClean`: If true, cleans all textures; if false, only cleans textures with reference count of 0. Default false.

Return value: Number of textures cleaned.

```typescript
//Usage example
// Clean unused textures (reference count is 0)
const cleaned = atlasManager.cleanupUnusedTextures();
console.log(`Cleaned ${cleaned} unused textures`);

// Force clean all textures
const total = atlasManager.cleanupUnusedTextures(true);
console.log(`Force cleaned ${total} textures`);

// Clean when switching scenes
function onSceneSwitch() {
    atlasManager.cleanupUnusedTextures();
}
```

---

#### Clearing Atlas

```typescript
//Interface
clear(): void
```

Clears all textures and releases large texture resources. Will clean up RenderTarget, be aware of reference relationships.

```typescript
//Usage example
// Clear atlas
atlasManager.clear();
console.log("Atlas cleared");
```

---

#### Destroying Manager

```typescript
//Interface
destroy(): void
```

Completely destroys the manager and releases all resources. The manager cannot be used after destruction.

```typescript
//Usage example
// Destroy manager
atlasManager.destroy();
console.log("Manager destroyed");
```



## III. Usage Recommendations

1. **Set Appropriate Atlas Size**
   Choose suitable `largeTextureSize` based on target devices:
   - Mobile devices: 1024x1024 or 2048x2048
   - PC/WebGL: 2048x2048 or 4096x4096
   - Note the maximum texture size limit of devices

2. **Enable Duplicate Checking**
   Keep `checkDuplicate=true` to avoid wasting space by adding duplicate textures.

3. **Extension Settings**
   If textures have scaling or rotation operations,it is recommended to set `extendSize` set to 1-2 to prevent sampling bleeding. If textures won't be scaled or rotated, set to 0 for better performance.

4. **Batch Operations**
   Use batch APIs like `addTextures` instead of looping through single API calls for better efficiency.

5. **Regular Cleanup**
   Call `cleanupUnusedTextures()` when switching scenes or reorganizing resources to release space.

6. **Monitor Usage Rate**
   Monitor atlas usage rate through `getStatistics()`. When usage rate approaches 100%, consider increasing `maxLargeTextures`.

7. **Proper Use of immediately**
   Usually keep `immediately=false` to let merge operations be delayed to idle time; only set to true when synchronous completion is necessary.

8. **Texture Unit Size Settings**
   `textureUnitSize` should be a common divisor of frequently used texture sizes. For example, if textures are mostly 32, 64, or 128 pixels, setting it to 16 or 32 is appropriate.

9. **Auto Replacement Choice**
   - If you want transparent atlas usage, set `autoReplace=true`
   - If you need precise control over replacement timing (e.g., batch replacement), set `autoReplace=false` and manually call replacement methods

10. **Atlas Quantity Control**
    `maxLargeTextures` should not be too large, as each large texture will occupy RenderTarget resources. It's recommended to set it based on actual needs; usually 4-8 is sufficient.



