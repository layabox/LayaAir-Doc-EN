# Summary of differences between 2.0 engine developers using 3.0

> Author: Tanishu && Charley

> [!Note]
> This document is only suitable for developers of old versions of the engine before 3.0. They are familiar with some project differences and will be further summarized later.

## 1. LayaAir3.0 Loader modification

### 1.1 Load a resource

Example:

```typeScript
var url = "xxx.png";
var type = Laya.Loader.IMAGE;
Laya.loader.load(url).then((res)=> {
	//Without type, used for regular resources
});

Laya.loader.load(url, type).then((res)=> {
	//With type, used to distinguish resources with different functions of the same suffix.
	//For example, the picture xxx.png is defined as TextureCube, use load("xxx.png", Laya.Loader.TEXTURECUBE). What you get is a TextureCube.
});
```

### 1.2 Load multiple resources (use array)

```typescript
var url1 = "xxx.png";
var url2 = "xxxxx.png";
var type1 = Laya.Loader.IMAGE;
var type2 = Laya.Loader.TEXTURE2D;
Laya.loader.load([url1, url2]).then((res:Array<any>)=> {
	//Load multiple, without type
});                 	 
Laya.loader.load([url1, url2], type).then((res:Array<any>)=> {
  //Load multiple and set the type uniformly
});                       	 
Laya.loader.load([{ url:url1, type: type1 }, { url:url2, type: type2 }]).then((res:Array<any>)=> {
    //Load multiple and set types separately
});
```

### 1.3 Load multiple files (combination)

```typeScript
let tasks:Array<Promise<any>> = [];
tasks.push(Laya.loader.load(url));
tasks.push(Laya.loader.load(url2));

Promise.all(tasks).then((res:Array<any>)=> {
	//for asynchronous loading
});
```

#### 1.4 Problems with Texture and Texture2D

The same resource address, whether loading Texture or Texture2D, has only one copy in the memory, but different types can be obtained.

```typescript
Laya.loader.load("1.png").then((res)=> { /* res是Texture */ });
Laya.loader.load("1.png", Loader.Texture2D).then((res)=> { /* res是Texture2D */ });

Laya.loader.getRes("1.png"); //res是Texture
Laya.loader.getRes("1.png", Loader.Texture2D); //res是Texture2D
Laya.Loader.getTexture2D("1.png"); //res是Texture2D
```

### 1.5 Load HTMLImage

```typescript
Laya.loader.fetch("1.png", Laya.Loader.IMAGE).then((res)=> { /* res是HTMLImage */ });
```

> [!Type|label:Tips]
>
> Cannot use Loader.getRes to obtain fetch results because the fetch method does not cache

### 1.6 Using Options

```typescript
Laya.loader.load(url, { group:xx, priority:1 }); //priority is not limited to 0-5. It is any integer. The larger the number, the higher the priority.
```

### 1.7 Prefab/scene issues

Loading these three files `lh/ls/gltf` will download and load all dependent resources.

But the node will not be created automatically. The cache is not a node either.

```typescript
Laya.loader.load("1.lh").then(res=> {
	/* Note that res is not a node type! You don't need to care about the type, you just need to know that it has a create method to instantiate the node tree. */
    	let node = res.create();
});
```

### 1.8 Compatibility issues between load and create in old versions of engines

Engines before 3.0 had two methods: Laya.loader.load() and Laya.loader.create(), which were used to load 2D and 3D resources respectively.

The 3.0 engine version can use the load() method uniformly.

For resources such as lh/ls/gltf, the create method of the old version is equivalent to the load+createNodes of the 3.0 engine.

For other resources, there is no difference between the create method and the load method. The create method of the old engine has been canceled in the 3.0 engine. Improper use of this method will cause memory leaks, so a compilation error needs to be reported to force developers to modify it.

### 1.9 Extend Loader capabilities

The original parseMap and createMap have been cancelled.

Write a class to implement the IResourceLoader interface, for example the simplest implementation:

```typescript
class MyLoader {
	load(task:ILoadTask) {
    	return task.loader.fetch(task.url, "json", task.createCallback()).then(data=> {
        	let obj = /*parse data*/;
        	return obj;
    	});
	}
}
```

There is no need to consider whether to load individually or batch load one of the links in the loading class, because task.createCallback can normalize the overall progress to 0~1.

For complex examples, please refer to TextureLoader/MaterialLoader/MeshLoader in the engine.

Then use Loader.registerLoader to register this class. For example

```typescript
Loader.registerLoader(["xyz"], YourLoader);
```



## 2. Instructions on dynamically loading resources in the IDE

No matter what resource is loaded, whatever the file name is in the editor, fill in the path. No need to put it in the bin directory, put it directly in the assets directory, with assets as the root path.

For example:

1. After dragging in FBX or GLTF, use load("xxx.FBX", Laya.Loader.HIERARCHY)

Load fbx or gltf in the editor. No need to manually create lh yourself. The editor automatically uses the converted results.

2. To load the blueprint shader, use load("xxxx.lbp") instead of "xxx.shader".



## 3. LayaAir3.0 input processing module modification

The original MouseManager and KeyboardManager were merged into InputManager. MouseManager was rarely used directly in games before, so it has little impact. KeyboardManager originally had only one interface, hasKeyDown, but now it can be changed to call InputManager.hasKeyDown.

Features of the new input processing system are:

### 3.1 2D and 3D unified interface,

Input can be processed through event listening and Laya.Script named functions. For example:

```typescript
this.aNode.on(Laya.Event.CLICK, ()=> {
console.log("clicked");
});

class MyScript extends Laya.Script {
	//Script incident
	onMouseClick(e:Event) {
    	console.log("clicked");
	}
}
aNode.addComponent(MyScript);
```

The above two methods are equivalent and can be used normally in both pure 2D or 2D/3D hybrid situations.



### 3.2 2D can correctly block 3D

During input processing, 2D can correctly block 3D.

### 3.3 Added MOUSE_DRAG and MOUSE_DRAG_END

Two new events have been added: MOUSE_DRAG and MOUSE_DRAG_END.

When the mouse is pressed and moved on an object (whether it is above the object or not), MOUSE_DRAG will continue to be dispatched to the object.

Dispatches MOUSE_DRAG_END to this object when the mouse is released (whether over it or not).

### 3.4 Removed RIGHT_MOUSE_DOWN and RIGHT_MOUSE_UP

 RIGHT_MOUSE_DOWN and RIGHT_MOUSE_UP were deleted and MOUSE_DOWN and MOUSE_UP were dispatched instead. The left, middle and right mouse buttons can be distinguished through Event.button.

### 3.5 Event Summary

The relevant input processing functions in Laya.Script are:

```typescript
/**
 	* Executed when mouse is pressed
 	*/
	onMouseDown?(evt: Event): void;

	/**
 	* Executed when the mouse is raised
 	*/
	onMouseUp?(evt: Event): void;

	/**
 	* Executed when the mouse moves on the node
 	*/
	onMouseMove?(evt: Event): void;

	/**
 	* Executed when the mouse enters the node
 	*/
	onMouseOver?(evt: Event): void;

	/**
 	* Executed when the mouse leaves the node
 	*/
	onMouseOut?(evt: Event): void;

	/**
 	* After the mouse is pressed on an object, it is executed when dragging
 	*/
	onMouseDrag?(evt: Event): void;

	/**
 	* Press and hold an object with the mouse, drag it a certain distance, and execute after releasing the mouse button.
 	*/
	onMouseDragEnd?(evt: Event): void;

	/**
 	* Executed when mouse clicks
 	*/
	onMouseClick?(evt: Event): void;

	/**
 	* Executed when double-clicking the mouse
 	*/
	onMouseDoubleClick?(evt: Event): void;

	/**
 	* Executed when right-clicking the mouse
 	*/
	onMouseRightClick?(evt: Event): void;

	/**
 	* Executed when the keyboard is pressed
 	*/
	onKeyDown?(evt: Event): void;

	/**
 	* Executed when the keyboard generates a character
 	*/
	onKeyPress?(evt: Event): void;

	/**
 	* Executed when the keyboard is raised
 	*/
	onKeyUp?(evt: Event): void;
```

Compared with the old version, there are the following changes

1) onClick name changed to onMouseClick

2) onDoubleClick name changed to onMouseDoubleClick

3) Removed onStageMouseDown, onStageMouseUp, onStageClick, onStageMouseMove

4) Removed onMouseEnter, which is duplicated with onMouseOver.





## 4. LayaAir3.0 component system modification

4.1 There are three component base classes in the component system of LayaAir2, Component, Script and Script3D.

1. LayaAir3.0 merges Scirpt and Script3D, that is, only Script can be used. Script3D can still be used, but it is just an alias for Script. Script can be mounted to 2D objects or 3D objects.

4.2 In LayaAir2, the difference between Component and Script is that Component mainly writes logic through inheritance, while Script is a relatively pure component mechanism.

1. In LayaAir3.0, Component has a complete life cycle, that is, onAwake, onStart, onEnable, onUpdate, onLateUpdate, onDisable, onDestroy, and underscore functions such as _onEnable, _onDisable are no longer used. _onEnable, _onDisable and other offline functions are still for internal use

2. Compared with Component, Script adds interactive behaviors, namely onTriggerEnter, onCollisionEnter, onMouseClick, onKeyDown and other callbacks related to input and output. Apart from this, there is no essential difference between Script and Component. Developers generally use Script.

4.3 Whether the Update/LateUpdate method of Component or Script runs in the IDE editing mode is determined by their runInEditor attribute. Developers generally implement this by attaching the @runInEditor decoration to the Script. Not running by default.

Regarding the use of modifiers, refer to the following example:

```typescript
const { regClass, property } = Laya;
//With @regClass(), it will be recognized as a script script class
@regClass()
export class Script extends Laya.Script {
	//Only if there is @property() on the property will it be recognized as an exposeable property in the IDE. As long as each property needs to be exposed, @property() needs to be added to it.
	@property( { type : String } )
	public text: string = "";

	constructor() {
    	super();
	}
}
```





## 5. Differences in the use of Runtime

The 3.0 scenario is completely different from the 2.0 concept.

The runtime of 3.0 can only be set on the 2D root node Scene2D on the scene or the root node of the prefab. Other child nodes no longer support runtime. If there is a need for code, a script must be used to implement it.

The UI of 3.0 does not have the var attribute. Name and var are merged. By default, only name is required. If you need to access the runtime through this.xxx on the scene inheritance class, check the Declare Var after name to export the attribute to the scene. in the base class.

3.0 no longer generates scene classes uniformly into one file, but generates them into the same-level directory of the Runtime class. When a scene class is specified for the runtime, the same-level directory of the scene class will automatically generate a file with the same name but the suffix Different base classes. The suffix name of the scene class base class is xxx.generated.ts

If the developer does not want to see the .generated.ts file, he can add the .generated.ts suffix to the vscode configuration file to block the generated base class.



## 6. 2D animation

2D animation no longer supports the ani format, and is used in combination with state machines and animation files.






