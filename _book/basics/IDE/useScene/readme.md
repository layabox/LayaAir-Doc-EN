# Scene management

`Laya.Scene` scene class, responsible for scene creation, loading, destruction and other functions

After the scene is removed from the node, it will not be recycled by the automatic garbage mechanism. If you want to recycle, please call the destroy interface. You can view the list of scenes that have not been destroyed through the `unDestroyedScenes` attribute.

### Open scene

#### 1. Basic use

`Laya.Scene.open(url: string, closeOther: boolean = true, param: any = null, complete: Handler = null, progress: Handler = null)`

<img src="images/1.png" alt="image-20221103165707451" style="zoom:25%;" />   	 

(Figure 1) Create a scene named OpenScene and save it in the path uiDemo/page/OpenScene.ls

```
Laya.Scene.open("uiDemo/page/OpenScene.ls", false);
```

You can open the scene like this in the code without closing other scenes.

#### 2. Pass and receive parameters

<img src="images/3.png" alt=" " style="zoom: 28%;" />

(Figure 2) Create a scene named Msg, which will pass text when entering this scene.

```
Laya.Scene.open("uiDemo/Msg.ls", false, { "text": "There is no check option, please check it first" });
```

Pass parameters to the next scene through `Laya.Scene.open`, the data is `{ "text": "There is no check option, please check it first" }`

<img src="images/2.png" alt="image-20221103172903482" style="zoom: 33%;" />

(Figure 3) In the Msg scene, add the Runtime class. The `onOpened` method will accept the incoming parameters. `param.text` is "There is no check option, please check it first"

<img src="images/4.png" alt="image-20221103173414315" style="zoom:50%;" />

 (Figure 4) *Note: After the scene is opened, call this method (if there is a pop-up animation, it will be executed after the animation is completed)*



### Close scene

**1. Close the specified scene**

`Laya.Scene.close(url: string, name: string = "")`

Based on the address, close the scene (including the dialog box)

**2. Close the current scene**

`this.close()`

<img src="images/5.png" alt="image-20221103174431210" style="zoom: 33%;" />

(Figure 5) Using the Runtime method, you can easily use `this.close()` to close the scene

**3, close all scenes (excluding dialog**)

`Laya.Scene.closeAll()`

**4, life cycle method called after closing**

<img src="images/6.png" style="zoom: 50%;" />

(Figure 6) The Runtime class of the scene will be called when the scene is closed. The `onClosed()` method can be used to release various resources within the method.

### Scene loading page

设置 `Laya.Scene.setLoadingPage(loadPage: Sprite)`

Set the loading interface. The engine will delay opening the loading interface after calling the open method, and close the loading interface after the page is added to the stage.

显示 `Laya.Scene.showLoadingPage(param: any = null, delay: number = 500)`

Display the loading interface and open the parameters. If it is a scene, it will be passed to the `onOpened` method to delay the opening time. The default is 500 milliseconds.

隐藏 `Laya.Scene.hideLoadingPage(delay: number = 500)`

<img src="images/8.png" alt="image-20221105102225313" style="zoom:40%;" />

(Figure 7) Define a script code and add the Laya.Prefab attribute that may be dragged into the loadingScene

 <img src="images/9.png" alt="image-20221105102404282" style="zoom:30%;" />

(Figure 8) In the Scene.ls scene, hang up the script and drag it into the Loading.ls scene



### Destroy

Destroy resources that are not currently being used `Laya.Scene.gc()`

Destroy the scene (including dialog) according to the address `Laya.Scene.destroy(url: string, name: string = "")`

<img src="images/10.png" alt="image-20221105103515022" style="zoom:50%;" />

(Figure 9)

