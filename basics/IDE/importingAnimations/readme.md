

# Import and use of 3D animation



## LayaAir currently supports imported 3D animations

- ### **Skeleton Skin Animation**

- ### **Rigid body animation**

 

## 1. Skeleton skinning animation

Only relatively simple animations can be produced in LayaAir. If you want complex animations, such as animations of characters running, you must create them in external software and import them into LayaAir for use. Usually we need to use 3dMax software to make models and animations, and export them to Fbx format files. The following will introduce how to import 3D models and animations.

### 1. Import model files

Put Fbx into the Assets directory

<img src="images/1.png" alt="image-20221109105123759" style="zoom:50%;" />

(figure 1)

Open LayaAir IDE. The IDE will recognize the Fbx file and can expand the Fbx file. You will see the file information in Figure 2. Take_001 is the skeleton skin animation file that comes with Fbx and is also its own standby animation.

Drag Fbx into the Scene3D scene, you can see the model, but there are no textures and animations. Let’s first introduce how to add materials and textures to LayaMonkey.

<img src="images/2.png" alt="image-20221109110840280" style="zoom: 35%;" />

(figure 2)



### 2. Texture setting association

Place the texture file under Assets

<img src="images/3.png" alt="image-20221109111236856" style="zoom:48%;" />

(image 3)

You need to create a material file to configure textures and other information for the model.

<img src="images/4.jpg" alt="4" style="zoom:40%;" />

(Figure 4)

Click on the newly created Material file. You can see a lot of configuration information on the right. Here we only need to drag the texture file into the material file.

<img src="images/5.png" alt="image-20221109111633571" style="zoom:30%;" />

(Figure 5)

Drag the material file into the Materials of LayaMonkey's SkinnedMeshRenderer component

<img src="images/6.png" alt="image-20221109111714022" style="zoom:30%;" />

(Figure 6)

You can see that the model already has texture effects

<img src="images/7.png" alt="image-20221109111755944" style="zoom:39%;" />

(Figure 7)



### 3. Import and use of skeleton skin animation

Here's how to add animation to LayaMonkey

When the Fbx file is dragged into the Scene3D scene, the Animator component will be automatically added. If not, you can add the Animator component yourself and ensure the AlwaysAnimate mode.

<img src="images/8.png" alt="image-20221109111912543" style="zoom:33%;" />

(Figure 8)

At this point, we need to create an AnimationController file, which is the 3D animation state machine

<img src="images/9.jpg" alt="9" style="zoom: 40%;" />

(Figure 9)

Drag the newly created Animation Controller file into the Animator component

<img src="images/10.png" alt="image-20221109112315395" style="zoom: 33%;" />

(Figure 10)

After double-clicking the AnimatorController, drag the Take_001 animation file into the Animator window

<img src="images/11.png" alt="image-20221109112505899" style="zoom:32%;" />

(Figure 11)

Click the Scene window again and click on the animation file preview below to see the animation effect.

<img src="gif/2.gif" alt="2" style="zoom:33%;" />

(Figure 12)

When you run the IDE at this time, you can play LayaMonkey animations, or you can drag LayaMonkey into the Assets directory as a prefab for easy reuse or code implementation.

<img src="images/12.png" alt="image-20221109112834572" style="zoom:33%;" />

(Figure 13)



### 4. Use of multiple individual bone skinning animation files

In most cases, the Fbx file exported by a model containing animation will contain both mesh information and animation information, but there are also some cases where the Fbx file only needs to export animation information. For example, if there are many animations in the same model, only one mesh information is needed, and other animation information can be exported through separate model files (without mesh information). When making animations, you should consider the reuse of animations for similar models in the scene. For example, different humanoid characters might all use the same walking and running animations. Animations can be reused as long as the skeletal structure is kept consistent.

In Figure 14, taking girl as an example, we put the Fbx file without animation information and multiple Fbx files with only animation into Assets

<img src="images/19.png" alt="image-20221109115632939" style="zoom: 50%;" />

(Figure 14)

After creating the AnimatorController, drag the standby and running animations into the Animator window to modify the animation name.

<img src="images/20.png" alt="image-20221109115632939" style="zoom: 30%;" />

(Figure 15)

> Note: The idle animation is the default animation at this time

As shown in Figure 16, we can preview the standby and running animations separately, and set whether to loop.

<img src="gif/3.gif" alt="3" style="zoom:35%;" />

(Figure 16)

Multiple individual bone skinning files are set successfully, and individual animations can also be set to other models for reuse.



### 5. Code running example

With the following code, we can load the girl prefab in any scene, add it to the scene, and click the screen with the mouse to switch animations

```
import { MainBase } from "./Main.generated";
import KeyBoardManager = Laya.InputManager;
import Keyboard = Laya.Keyboard;
const { regClass, property } = Laya;

@regClass()
export class Main extends MainBase {

	private _animator: Laya.Animator;
	private _isRun: boolean;
	onAwake() {
    	console.log("Game start");
    	//Load the specified model prefab and add it to the Scene3D scene
    	Laya.loader.load("girl/girl.lh").then(res => {
        	let girl : Laya.Sprite3D = res.create();
        	this.scene3D.addChild(girl);
        	//Get Animator
        	this._animator = girl.getComponent<Laya.Animator>(Laya.Animator);
    	}); 	 
    	this.on( Laya.Event.MOUSE_DOWN, this, this.switchAni );
	}

	switchAni(): void {
    	if (this._isRun) {
   		 //Play the corresponding animation
        	this._animator.play("idle");           	 
    	} else {
   		 this._animator.play("run");
    	}  
    	this._isRun = !this._isRun;
	}    
}
```



## 2. Rigid body animation



### 1. Import glTF or fbx files

Put glTF into the Assets directory and open LayaAir IDE. The IDE will recognize the glTF file and expand the glTF file. You will see the file information in Figure 17, where Animation_0 is the rigid body animation file that comes with glTF.

 <img src="images/21.png" alt="image-20221109170859228" style=" zoom: 50%;" />

(Figure 17)

Drag glTF into the Scene3D scene, you can see the model, but there is no animation. Let’s first introduce how to use animation.

<img src="images/22.png" alt="image-20221109171201077" style="zoom:50%;" />

(Figure 18)

### 2. Import and use of animation

When the glTF file is dragged into the Scene3D scene, the Animator component will be automatically added. If not, you can add the Animator component yourself and ensure the AlwaysAnimate mode.

<img src="images/23.png" alt="image-20221109171352077" style="zoom:33%;" />

(Figure 19)

At this point, we need to create an AnimationController file, which is the 3D animation state machine

<img src="images/24.jpg" alt="9" style="zoom: 40%;" />

(Figure 20)

Drag the newly created Animation Controller file into the Animator component

<img src="images/25.png" alt="image-20221109171621289" style="zoom:33%;" />

(Figure 21)

After double-clicking the AnimatorController, drag the Take_001 animation file into the Animator window

 <img src="images/26.png" alt="image-20221109171729141" style="zoom:33%;" />

(Figure 22)

When you run the IDE at this time, you can play rigid body animation, or you can drag BoxAnimated into the Assets directory as a prefabricated body for easy reuse or code implementation.

<img src="gif/4.gif" alt="4" style="zoom: 35%;" />

(Figure 23)





