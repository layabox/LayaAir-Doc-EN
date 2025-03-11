# Reduce CPU usage

### **1. Reduce dynamic attribute lookup**

Any object in JavaScript is dynamic and you can add attributes at will. However, searching for an attribute among a large number of attributes can be time-consuming. If you need to use a certain attribute value frequently, you can use a local variable to save it:

```typescript
foo()
{
var prop=this.target.prop;
//use prop
this.process1(prop);
this.process2(prop);
this.process3(prop);
}
```

### 2. Timer

 LayaAir provides two timer loops to execute code blocks.

1. The execution frequency of `Laya.timer.frameLoop` depends on the frame frequency. The current frame rate can be viewed through Stat.FPS.


1. The execution frequency of `Laya.timer.loop` depends on the time specified by the parameter.

```typescript
Laya.timer.frameLoop(1, this, this.animateFrameRateBased);
Laya.stage.on("click", this, this.dispose);
dispose()
{
	Laya.timer.clear(this, this.animateFrameRateBased);
}
```

When the life cycle of an object ends, remember to clear its internal Timer:

### **3. How to obtain the boundary of the displayed object**

In relative layout, it is very often necessary to correctly obtain the bounds of the displayed object. There are many ways to get the boundaries of the displayed object, and it is necessary to know the differences.

1. Use getBounds/getGraphicBounds.

```typescript
var sp=new Laya.Sprite();
sp.graphics.drawRect(0,0,100,100,"#FF0000");
var bounds:Laya.Rectangle=sp.getGraphicBounds();
Laya.stage.addChild(sp);
```

 getBounds can meet most needs, but because it needs to calculate the boundary, it is not suitable for frequent calls.

1. Set the container's autoSize to true.

```typescript
var sp=new Laya.Sprite();
sp.autoSize=true;
sp.graphics.drawRect(0,0,100,100,"#FF0000");
Laya.stage.addChild(sp);
```

The above code can correctly obtain the width and height at runtime. autoSize will be recalculated when the width and height are obtained and the state of the display list changes (autoSize calculates the width and height through getBoudns). So it is not advisable to apply autoSize to a container with a large number of sub-objects. If size is set, autoSize will not take effect.

 Get the width and height after using loadImage:

```typescript
var sp=new Laya.Sprite();
sp.loadImage("res/apes/monkey2.png",0,0,0,0,Laya.Handler.create(this,function()
{
	console.log(sp.width,sp.height);  
}));
Laya.stage.addChild(sp);
```

 loadImage can correctly obtain the width and height only after the callback function after loading is triggered.

1. **Directly call the size setting:**

```typescript
Laya.loader.load("res/apes/monkey2.png",Laya.Handler.create(this,function()
{
  var texture=Laya.loader.getRes("res/apes/monkey2.png");
  var sp=new Laya.Sprite();
  sp.graphics.drawTexture(texture,0,0);
  sp.size(texture.width,texture.height);
  Laya.stage.addChild(sp);
}));
```

Using Graphics.drawTexture does not automatically set the width and height of the container, but you can use the width and height of the Texture to assign it to the container. Needless to say, this is the most efficient way.

**Note: getGraphicsBounds is used to obtain the width and height of vector drawing. **

### **4. Change the frame rate based on activity status**

 There are three modes of frame rate,

- Stage.FRAME_FAST

  In fast mode, the maximum FPS is the maximum frame rate of the monitor. If the maximum frame rate of the monitor is 60, then the maximum FPS is 60. If the maximum frame rate of the monitor is 120, then the maximum FPS is 120.

- Stage.FRAME_SLOW

  In slow mode, the maximum FPS is half of the monitor's maximum frame rate. During the game running, the engine will discard every other frame. If it can actually reach 40 frames, then the final frame rate of the game is only 20. If the frame rate can reach 100, then the final frame rate can only be 50.

- Stage.FRAME_MOUSE

  Mouse mode selectively switches between fast mode and slow mode. Sometimes the game does not need to be executed at full frame rate. For example, at 60 frames full frame, 30FPS can already meet the response of human vision in most cases, but mouse interaction At this time, 30FPS may cause incoherence in the picture, so Stage.FRAME_MOUSE came into being.

 The following example shows moving the mouse on the canvas at the frame rate of Stage.FRAME_SLOW so that the ball follows the movement of the mouse:

```typescript
Laya.init(this.Browser.width,this.Browser.height);
Laya.Stat.show();
Laya.stage.frameRate=Laya.Stage.FRAME_SLOW;

var sp=new Laya.Sprite();
sp.graphics.drawCircle(0,0,20,"#990000");
Laya.stage.addChild(sp);

Laya.stage.on(Laya.Event.MOUSE_MOVE,this,function()
{
  sp.pos(Laya.stage.mouseX,Laya.stage.mouseY);
});
```

![图片1.png](https://official.layabox.com/laya_data/Chinese/LayaAir_AS3/2D/advanced/PerformanceOptimization/CPU/img/1.png)

(figure 1)

 At this time, the FPS displays 30, and when the mouse moves, you can feel that the update of the ball position is inconsistent. Set Stage.frameRate to Stage.FRAME_MOUSE:

```typescript
Laya.stage.frameRate = Laya.Stage.FRAME_MOUSE;
```

![图片1.png](https://official.layabox.com/laya_data/Chinese/LayaAir_AS3/2D/advanced/PerformanceOptimization/CPU/img/2.png)

(figure 2)

 At this time, after moving the mouse, the FPS will display 60, and the screen smoothness will be improved. After the mouse remains motionless for 2 seconds, FPS will return to 30 frames.

### **5. Use callLater**

callLater delays the execution of the code block until before rendering of this frame. If the current operation frequently changes the state of an object, you may consider using callLater to reduce repeated calculations.

 Consider a figure for which setting any appearance-changing properties will cause the figure to be redrawn:

```typescript
var rotation=0,
scale=1,
position=0;

private function setRotation(value):void
{
  this.rotation=value;
  update();
}

private function setScale(value):void
{
  this.scale = value;
	update();
}
private function setPosition(value):void
{
	this.position = value;
	update();
}
public function update()
{
	console.log('rotation: ' + this.rotation + '\tscale: ' + this.scale + '\tposition: ' + this.position);
}
```

 Call the following code to change the status:

```
setRotation(90);
setScale(2);
setPosition(30);
```

 The console print result is:

```
rotation: 90scale: 1position: 0
rotation: 90scale: 2position: 0
rotation: 90scale: 2position: 30
```

update was called three times, and the final result was correct, but the first two calls were unnecessary.

 Try changing the three updates to:

```
Laya.timer.callLater(this, update);
```

At this time, update will only be called once, and it is the result we want.

### **6. Picture/Album Loading**

After completing the loading of images/atlases, the engine will start processing image resources. If an image gallery is loaded, each sub-image will be processed. If a large number of images are processed at one time, this process may cause lengthy delays.

In the resource loading of the game, resources can be loaded according to levels, scenes, etc. The better the picture is processed at the same time, the more responsive the game will be at that time. After the resource is used, it can also be unloaded to release memory.
