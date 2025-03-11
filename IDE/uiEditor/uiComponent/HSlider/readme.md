# Horizontal slider component (HSlider)

## 1. Create HSlider component through LayaAir IDE

The HSlider and VSlider components are both subclasses of the Slider component. They represent horizontal slide bars and vertical slide bars respectively. Users can select values ​​by moving the slider between slider tracks. Commonly used for example, player progress control, volume control, numerical adjustment on some UI, etc. The detailed properties of HSlider can be viewed in [API](https://layaair.com/3.x/api/Chinese/index.html?version=3.0.0&type=2D&category=UI&class=laya.ui.HSlider).

### 1.1 Create HSlider

As shown in Figure 1-1, you can right-click in the `Hierarchy' window to create it, or you can drag and drop from the `Widgets' window to add it.

<img src="img/1-1.png" alt="1-1" style="zoom:50%;" />

(Picture 1-1)

Sliders can be made of two or three parts. If it is two parts, it includes the basemap resource `hslider.png` and the slider resource `hslider$bar.png`, as shown in Figure 1-2. Resources should have at least these two, otherwise the sliding function cannot be implemented.

![1-2](img/1-2.png)

(Figure 1-2)

If it is a three-part slide bar, it includes the slider resource `hslider$bar.png`, the progress bar resource `hslider$progress.png`, and the basemap resource `hslider.png`, as shown in Figure 1-3. If the progress bar resource component is missing, no error will be reported, but the progress will not be displayed.

<img src="img/1-3.png" alt="1-3" style="zoom:80%;" />

(Figure 1-3)

> The progress bar resource `hslider$progress.png` can be interchanged with the basemap resource `hslider.png`. After the exchange, the progress can be displayed in reverse.

The HSlider component created by LayaAir by default is composed of two parts. As shown in the animation 1-4, the HSlider component adopts the horizontal direction. The slider track expands from left to right, and dragging the slider with the mouse will display numerical labels.

<img src="img/1-4.gif" alt="1-4" style="zoom:50%;" />

(Animation 1-4)



### 1.2 HSlider properties

The unique properties of HSlider are as follows:

![1-5](img/1-5.png)

(Figure 1-5)

| **Properties**	| **Function description**	|
| -------------- | ------------------------------------------------------------ |
| max        	| The value when the slider is dragged to the far right, the default value is 100	|
| min        	| The value when the slider is dragged to the far left, the default value is 0	|
| showLabel  	| Whether to display labels. The default is true. Dragging the slider during runtime will display the value.	|
| showProgress | Whether to display the progress bar. The default is false. If there is a progress bar resource `hslider$progress.png`, you can check this item |
| skin       	| Basemap resource for slider	|
| sizeGrid   	| Valid scaling grid data (nine-square grid data) of the slider basemap resource	|
| tick       	| The minimum unit of the slider scale value. The amount the value changes each time the slider is dragged. The default value is 1 |
| value      	| The current scale, that is, the current value of the slider, should be equal to max or min, or a value between them |
| allowClickBack | Whether to allow changing the value by clicking the slider. The default is false, which prohibits changing the value by clicking the slider. At this time, the only way to change the value is by dragging the slider. When it is true, you can click the target area of ​​the slider to quickly jump to the current scale and change the value |

After setting the value of attribute max of HSlider to 20, the value of attribute min to 0, and the value of attribute value to 5, the display effect is as follows:

![1-6](img/1-6.png)

(Figure 1-6)

Set the attribute showLabel to true, the attribute showProgress to true, and the attribute tick value to 3. The effect is as shown in the following animation:

![1-7](img/1-7.gif)

(Animation 1-7)

The progress bar resource `hslider$progress.png` can be interchanged with the basemap resource `hslider.png`. The effect is as follows:

![1-8](img/1-8.gif)

(Animation 1-8)



### 1.3 Script control HSlider

In the Scene2D property settings panel, add a custom component script. Then, drag the HSlider into its exposed property entry. You need to add the following sample code to implement script control of HSlider:

```typescript
const { regClass, property } = Laya;

@regClass()
export class NewScript extends Laya.Script {

	@property({ type: Laya.HSlider })
	public hslider: Laya.HSlider;

	//Execute after the component is activated. At this time, all nodes and components have been created. This method is only executed once.
	onAwake(): void {
    	this.hslider.pos(300, 300);//Slider position
    	this.hslider.skin = "resources/hslider.png";//slider basemap skin
    	this.hslider.value = 0.5;
    	this.hslider.max = 50;
    	this.hslider.min = 0;
    	this.hslider.tick = 1;
    	this.hslider.showProgress = true;//The hslider$progress.png resource must exist, otherwise an error will be reported
	}
}
```



## 2. Create HSlider component through code

Sometimes, you need to control UI components with code, create the UI_HSlider class, and set the HSlider through code. The following example uses code to create an HSlider component and outputs its value on the console.

Sample code:

```typescript
const { regClass, property } = Laya;

@regClass()
export class UI_HSlider extends Laya.Script {

	constructor() {
    	super();
	}

	onAwake(): void {
    	let skins: any[] = [];
    	skins.push("hslider.png", "hslider$bar.png"); //Image resources come from "Engine API Usage Example"
    	Laya.loader.load(skins, Laya.Handler.create(this, this.placeHSlider));
	}

	private placeHSlider(): void {
    	let hs: Laya.Slider = new Laya.HSlider();
    	hs.skin = "hslider.png";

    	hs.width = 300;
    	hs.pos(50, 500);
    	hs.min = 0;
    	hs.max = 100;
    	hs.value = 50;
    	hs.tick = 1;

    	hs.changeHandler = new Laya.Handler(this, this.onChange);
    	this.owner.addChild(hs);
	}

	private onChange(value: number): void {
    	console.log("Slider position: " + value);
	}
}
```

running result:

![2-1](img/2-1.gif)

(Animation 2-1)

