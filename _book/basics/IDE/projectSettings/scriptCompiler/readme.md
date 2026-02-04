# Script Compilation

> Author: Charley

Script compilation configuration controls **TypeScript compilation behavior** and **release version code processing**.

Overall divided into two parts:

- **Script Compilation Options**: Affect compilation phase entry, dependencies, macro definitions, etc.
- **Release Version Options**: Only effective during release build, used for code stripping, cleaning, and compression

## 1. Script Compilation Options

### 1.1 Macro Definitions (`defines`)

`defines` is used to inject conditional variables at compile time to execute different code logic in different environments (debug / release). The most common use is distinguishing debug code from release code.

In code, you can directly use these macro variables through conditional judgment, for example:

```typescript
if (DEBUG) {
    // Debug logic
}
```

**Configuration Description**

- **Purpose**: Define compile-time variables to distinguish debug environment from release environment
- **Type**: Array
- **Structure**: Each element contains the following fields
  - `name` (Name): Variable name, default `"DEBUG"`
  - `debugValue` (Debug Value): Value in debug mode, default `"true"`
  - `releaseValue` (Release Value): Value in release mode, default `"false"`

### 1.2 External Packages (`external`)

`external` is used to declare external modules or SDKs that the project depends on. These modules **won't be packaged into the final output** during compilation and release but remain as external references.

This configuration is typically used for:

- Introducing libraries provided by the runtime environment
- Using modules injected via CDN or host environment

**Configuration Description**

- **Purpose**: Specify external modules not participating in packaging
- **Type**: String Array
- **Usage**: Add module names directly to the array

### 1.3 Package Aliases (`alias`)

`alias` is used to set aliases for module paths to simplify import paths, improve code readability, and avoid deep relative paths.

This configuration is typically used together with the `paths` configuration in `tsconfig.json`.

**Configuration Description**

- **Purpose**: Set aliases for module paths
- **Type**: Array
- **Structure**: Each element contains
  - `oldName` (Old Name): Original path
  - `newName` (New Name): Replaced path

### 1.4 Character Encoding (`charset`)

`charset` is used to specify the character encoding format of script files. In most projects, recommend using `utf8` to support various international characters.

**Configuration Description**

- **Purpose**: Specify script file encoding format
- **Type**: String
- **Options**: `"ascii"` / `"utf8"`
- **Default**: `"utf8"`

### 1.5 Compilation Entry Files (`entries`)

`entries` is used to specify the **compilation entry file list**.

These files will be treated as compilation starting points by the compilation tool and generate corresponding export statements during compilation to ensure they can be correctly compiled and referenced.

`entries` is applicable to **multi-entry or modular compilation scenarios**.

**Configuration Description**

- **Purpose**: Specify compilation entry files
- **Type**: String Array
- **Content**: TypeScript file paths

**Usage Recommendations**

- Only add files that need to be independent module entry points
- Avoid adding unnecessary entry files to avoid increasing compilation time and output size
- In modular projects, configure an entry file for each major module

### 1.6 Project Main Script (`mainScript`)

`mainScript` is used to specify the **project runtime main startup script**.

It can replace the default startup scene and allow developers to control initialization logic and startup timing before scene loading.

Unlike entries, `mainScript` receives **special processing** after compilation and directly participates in LayaAir's startup process.

**Configuration Description**

- **Purpose**: Specify the project's main startup script
- **Type**: String
- **Content**: Points to a TypeScript file (such as `Entry.ts`)

**Usage Recommendations**

- Always set `mainScript` to `Entry.ts`

- Ensure the file contains a standard startup structure

  ```typescript
  //This script must be an async main function, cannot modify naming
  export async function main() {
      console.log("Hello LayaAir!");

      //Example: Load scene and open scene
      Laya.Scene.open('Scene.ls');
  }
  ```

- Avoid writing test code in the main function, keep startup logic concise

- Follow LayaAir Engine startup process specifications to ensure stable project operation

> It's important to note that startup script execution only takes effect **after release**, or when preview mode is **startup scene** (not **current scene**).

### 1.7 Difference Between entries and mainScript

Although both entries and mainScript relate to "entries," their purpose levels and usage scenarios are completely different.

**Configuration Form**

- entries: String array, can specify multiple compilation entries
- mainScript: Single string, can only specify one startup script

**Effective Phase**

- entries: Affects **compilation phase**, determines which files serve as compilation entry points
- mainScript: Affects **runtime phase**, determines which script the project starts executing from

**Compilation Processing**

- entries: Compilation tool generates export statements for each entry, ensuring participation in compilation
- mainScript: Compilation tool performs special processing on this file and integrates it with engine startup process

**Relationship with Engine**

- entries: Only compilation tool configuration, no direct relationship with engine
- mainScript: Directly participates in LayaAir Engine startup mechanism

**Execution Mechanism**

- entries: Not automatically executed, needs to be triggered through module references
- mainScript: Automatically loaded and its `main` function executed by the engine at project startup

## 2. Release Version Options

Release version options only take effect during **release build**, used to control code stripping, debug information cleanup, and compression behavior.

### 2.1 Keep Unused Component Scripts (`keepUnusedComponentScripts`)

Default value is `false`.

When this option is enabled, even if certain component scripts are not referenced in the scene, they will be included in the release version.

Applicable for scenarios that need to dynamically load components through code.

### 2.2 Drop Debugger Statements (`dropDebugger`)

Default value is `false`.

When enabled, all `debugger` statements in the release version will be removed.

### 2.3 Drop Console Statements (`dropConsole`)

Default value is `false`.

When enabled, console output statements such as `console.log` and `console.warn` in the release version will be deleted, helping to reduce irrelevant output and optimize size.

### 2.4 Minification (`minify`)

Used to control code compression behavior for the release version.

#### 2.4.1 Keep Names (`keepNames`)

Default value is `false`.

When enabled, compressed code will retain class and function names for easier debugging of stack information, but will slightly increase size.

#### 2.4.2 Keep Unused Code (`keepUnused`)

Default value is `false`.

When enabled, even if certain functions or variables are unused in the code, they will be retained in the compressed code.
