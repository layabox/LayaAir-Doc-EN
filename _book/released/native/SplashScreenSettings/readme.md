# Splash Screen Settings

This document introduces how to configure the application splash screen in LayaNative projects, including both IDE visual settings and manual configuration file modification methods.

## 1. IDE Settings

In LayaAir IDE, you can intuitively configure the splash screen through the project settings interface.

![IDE Splash Screen Settings](ide.jpg)

### Configuration Steps
1. Open LayaAir IDE.
2. Find and open **Project Settings** in the top menu bar or toolbar.
3. Select **Splash Screen** in the left navigation bar.
4. Make the following configurations in the right panel:

### Parameter Description

*   **Enable**: Check this option to enable the splash screen function.
*   **Background Color**: Click the color block to select the background color of the splash screen.
*   **Image**: Set the image resource path for the splash screen (for example: `bin/splash.png`). Click the folder icon to browse and select the file.
*   **Fit Mode**: Select the image display fit mode.
    *   **Center**: Display at original size centered.
    *   **Contain**: Scale maintaining aspect ratio to ensure the image displays completely within the screen.
    *   **Cover**: Scale maintaining aspect ratio to fill the entire screen (may crop edges).
    *   **Fill**: Stretch the image to fill the screen (may deform).
*   **Min Display Time**: Set the minimum display time for the splash screen (unit is usually seconds). Adjust by sliding the slider or directly entering a value.
*   **Allow Enable in Preview**: After checking, the splash screen will also be displayed during preview runtime within the IDE.

---

## 2. Manual Settings (config.ini)

Besides setting in the IDE, you can also directly modify the `[waterMark]` node in the `config.ini` file in the publish directory to configure the splash screen. This is very useful when fine-tuning published packages or automating builds.

**Configuration File Location**: `config.ini` in the published resource directory (for example `publish/windows/resource/config.ini`)

### Parameter Details

| Parameter Name | Type | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Enabled** | Boolean | `true` | Whether to enable splash screen function.<br>`true`: Enable<br>`false`: Disable |
| **BackgroundColor** | String | `'#000000'` | Background color, supports hexadecimal color string. |
| **Image** | String | `image/splash.png` | Path to the image file (relative to resource root directory). |
| **FitMode** | String | `Center` | Image fit mode, corresponding to IDE options:<br>- **Center** (Centered)<br>- **Contain** (Scaled)<br>- **Cover** (Cover)<br>- **Fill** (Stretched) |
| **Duration** | Integer | `5000` | Display duration. Note that the unit here is **milliseconds (ms)**. |
| **PositionX** | Float | `0.5` | Horizontal position percentage of image center point (0.0 ~ 1.0).<br>`0.5` is horizontally centered. |
| **PositionY** | Float | `0.3` | Vertical position percentage of image center point (0.0 ~ 1.0).<br>`0.3` means located at 30% down from the top. |

### Configuration Example

```ini
[waterMark]
Enabled=true
BackgroundColor='#000000'
Image=image/splash.png
FitMode=Center
Duration=5000       #5000ms = 5 seconds
PositionX=0.5
PositionY=0.3
```
