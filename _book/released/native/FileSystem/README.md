# conch.readFileFromAsset / conch.isFileExistsInAsset Usage Instructions

These two interfaces are used to **read resource files packaged with the application (Asset/in-package resources)**. Typical uses include reading JS scripts under `scripts/`, `config.ini`, images/fonts, and other resources.

> Note: They read "in-package resources," not downloaded cache directories or writable local storage directories. For writable directories, refer to interfaces like `conch.getCachePath()` / `conch.getLocalStoragePath()`.

## 1. API Definitions

### 1.1 conch.isFileExistsInAsset(file)

- **Signature**: `conch.isFileExistsInAsset(file: string): boolean`
- **Parameters**
  - **file**: Resource relative path (e.g., `"scripts/apploader.js"`)
- **Return Value**
  - **true**: Resource exists
  - **false**: Resource doesn't exist, or Asset resource reader not initialized on current platform

### 1.2 conch.readFileFromAsset(file, encode)

- **Signature**: `conch.readFileFromAsset(file: string, encode: string): string | ArrayBuffer | null`
- **Parameters**
  - **file**: Resource relative path (e.g., `"config.ini"`, `"image/splash.png"`)
  - **encode**: Encoding/reading mode
    - Pass **`"utf8"`**: Returns **string**
    - Pass any other string (**`"raw"`** is commonly used in projects): Returns **ArrayBuffer** (binary)
- **Return Value**
  - Read successful: `string` or `ArrayBuffer`
  - Read failed: `null` (e.g., file doesn't exist, or Asset reader not initialized)

## 2. JS Usage Examples

### 2.1 Check if File Exists

```javascript
const ok = conch.isFileExistsInAsset("scripts/runtimeInit.js");
console.log("exists:", ok);
```

### 2.2 Read Text (UTF-8)

```javascript
const text = conch.readFileFromAsset("config.ini", "utf8");
if (text == null) {
  throw new Error("config.ini not found in asset");
}
console.log(text);
```

### 2.3 Read Binary (ArrayBuffer)

```javascript
const ab = conch.readFileFromAsset("image/splash.png", "raw");
if (ab == null) {
  throw new Error("image/splash.png not found in asset");
}
const bytes = new Uint8Array(ab);
console.log("png bytes:", bytes.length);
```

### 2.4 Typical Usage: Read Script from Asset and Write to Cache Directory

Example usage from the project (see `conch/src/network/tests/UploadFileTestClient.js`):

```javascript
var data = conch.readFileFromAsset("scripts/async.js", "raw");
var path = conch.getCachePath() + "/async.js";
fs_writeFileSync(path, data);
```

## 3. Path Rules (Very Important)

- **Always use relative paths**, do not start with `/`.
- **Recommend using `/` as separator**, e.g., `scripts/index.js` (consistent across platforms).
- **Keep case consistent**: Android/OHOS resource paths are usually case-sensitive, Windows usually is not. To avoid cross-platform issues, please unify case.

## 4. Asset Read Directories by Platform (Resource Root Directory)

The `file` parameter is a path relative to the "resource root directory." The table below describes where the resource root directory is on each platform and where it's typically placed in published projects.

### 4.1 Android

- **Resource Root Directory**: APK's `assets/` root directory
  - Published project example: `publish/android_studio/app/src/main/assets/`
  - Example: `conch.readFileFromAsset("scripts/apploader.js", ...)` corresponds to `assets/scripts/apploader.js`
- **Expansion Pack (OBB/ZIP)**: If `apkExpansionMainPath` / `apkExpansionPatchPath` is configured, read order is:
  - First check APK `assets/`
  - Then check Expansion Main
  - Then check Expansion Patch

### 4.2 iOS

- **Resource Root Directory**: Resource root path within App Bundle (provided by native layer `CToObjectCGetRootAssetsPath()`)
  - Published project example: `publish/ios/resource/` (usually copied into `.app` package)
  - Example: `conch.readFileFromAsset("scripts/apploader.js", ...)` corresponds to `.../resource/scripts/apploader.js` within the App package (specific absolute path determined by system/packaging method)

### 4.3 OHOS (HarmonyOS)

- **Resource Root Directory**: `resources/rawfile/` root directory (accessed via `OH_ResourceManager_OpenRawFile`)
  - Published project example: `publish/ohos/entry/src/main/resources/rawfile/`
  - Example: `conch.readFileFromAsset("scripts/apploader.js", ...)` corresponds to `rawfile/scripts/apploader.js`

### 4.4 Windows

- **Resource Root Directory**: Directory where the executable file (EXE) is located (`OS::getAssetRootPath()` returns `parent(exePath)`)
  - Example: `conch.readFileFromAsset("scripts/apploader.js", ...)` corresponds to `<exe_dir>/scripts/apploader.js`
  - In published projects, if resources are placed in a subdirectory (e.g., `publish/windows/resource/scripts/...`), you need to ensure the runtime resource root directory matches:
    - Common practice is to **copy resources like `scripts/`, `config.ini`, `image/` to the same directory as the EXE**, or place the EXE in the `resource/` directory to run

### 4.5 Linux

- **Resource Root Directory**: Directory where the executable file is located (same as Windows)
  - Example: `conch.readFileFromAsset("scripts/apploader.js", ...)` corresponds to `<exe_dir>/scripts/apploader.js`

## 5. Behavior Details (For Troubleshooting)

- **Failure Return Values**:
  - `readFileFromAsset(...)` returns `null` on failure
  - `isFileExistsInAsset(...)` returns `false` on failure
- **One-time Read into Memory**: `readFileFromAsset` reads the entire file into memory (returns string or ArrayBuffer). For large files, please pay attention to memory usage.
