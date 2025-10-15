# Video Node (VideoNode)

## 1. Using the Video Node in LayaAir IDE

### 1.1 Creating a VideoNode

As shown in Figure 1-1, you can create a **VideoNode** by right-clicking in the **Hierarchy** window, or by dragging it from the **Widget** panel into the scene.

<img src="img/1-1.png" alt="1-1" style="zoom:50%;" />

(Figure 1-1)

### 1.2 Property Overview

After adding a **VideoNode** to the scene view, its dedicated properties appear in the property panel, as shown in Figure 1-2.

![1-2](img/1-2.png)

(Figure 1-2)

| Property            | Description                                                                                                                                                   |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **source**          | The video source.                                                                                                                                             |
| **mode**            | Playback mode — there are two options: *player* and *decoder*. If the selected mode is unsupported, the other mode will be used automatically. Details below. |
| **autoplay**        | Whether the video plays automatically when added to the stage. Default is `false`.                                                                            |
| **allowBackground** | Whether background playback is allowed. Default is `false`.                                                                                                   |
| **loop**            | Whether the video loops. Default is `false`.                                                                                                                  |
| **muted**           | Mutes the video when enabled. Default is `false`.                                                                                                             |
| **controls**        | Whether to show the video’s control UI. Default is `false`.                                                                                                   |
| **underGameView**   | Whether to display the video beneath the game canvas. Default is `false`. (The canvas must be transparent for this option to work.)                           |
| **objectFit**       | The video scaling mode. Options: `fill`, `contain`, `cover`. Default is `contain`. Details below.                                                             |

### 1.3 Playback Mode (mode)

The two playback modes behave differently:

When using **decoder mode**, the video is captured as a texture, matching the node’s width and height, as shown in Animation 1-3.

![1-3](img/1-3.gif)

(Animation 1-3)

When using **player mode**, the video floats above the main canvas and maintains its original aspect ratio, as shown in Animation 1-4.

![1-4](img/1-4.gif)

(Animation 1-4)

### 1.4 Scaling Mode (objectFit)

The **objectFit** property defines how the video scales within its container:

* **fill**: The video stretches to completely fill the container, ignoring its original aspect ratio. (Figure 1-5)

![1-5](img/1-5.png)

(Figure 1-5)

* **contain**: Preserves the original aspect ratio so that the video fits entirely within the container, possibly leaving empty space. (Figure 1-6)

![1-6](img/1-6.png)

(Figure 1-6)

* **cover**: Preserves the original aspect ratio while ensuring the video covers the container. Parts of the video may be cropped. (Figure 1-7)

![1-7](img/1-7.png)

(Figure 1-7)

### 1.5 Controlling VideoNode via Script

After assigning a video file to the **Source** property (as described in section 1.2), the video will not play automatically. You must control it through code.
In the **Scene2D** property panel, add a custom component script and drag the **VideoNode** into its exposed property slot.
Example:

```typescript
const { regClass, property } = Laya;

@regClass()
export class NewScript extends Laya.Script {

    @property({ type: Laya.VideoNode })
    public video: Laya.VideoNode;

    constructor() {
        super();
    }

    // Executes once after all nodes and components are created
    onAwake(): void {
        // Play video when user clicks
        Laya.stage.on(Laya.Event.MOUSE_DOWN, () => {
            Laya.loader.load("resources/layaAir.mp4").then(() => {
                this.video.play(); // Play video
            });
        });
    }
}
```

When running inside the **LayaAir IDE**, the **VideoNode** can play automatically.
However, in browsers such as **Chrome**, automatic playback is only allowed if **muted autoplay** is enabled.
To play with sound, user interaction (e.g., click or tap) is required.

You can also control the video’s texture update rate through code, as some browsers have different playback policies:

```typescript
const { regClass, property } = Laya;

@regClass()
export class NewScript extends Laya.Script {

    @property({ type: Laya.VideoNode })
    public video: Laya.VideoNode;

    onEnable(): void {
        // Play video on click
        Laya.stage.on(Laya.Event.MOUSE_DOWN, () => {
            // Set video texture update rate
            this.video.videoTexture.useFrame = true;
            this.video.videoTexture.updateFrame = 30;

            this.video.play();
        });
    }
}
```

## 2. Creating a VideoNode via Code

If you don’t want the **VideoNode** to appear on stage at startup, you can create it dynamically in code.
Add a custom script in **Scene2D**, for example:

```typescript
const { regClass, property } = Laya;

@regClass()
export class NewScript extends Laya.Script {

    constructor() {
        super();
    }

    // Executes once when all nodes and components are ready
    onAwake(): void { 
        let video = new Laya.VideoNode();
        // Add to stage
        Laya.stage.addChild(video);
        video.pos(200, 200); // Set position
        video.source = "resources/layaAir.mp4"; // Set video source
        video.play(); // Start playback
    }
}
```
