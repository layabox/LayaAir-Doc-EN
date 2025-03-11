# Timer

Timer `Laya.Timer` is a clock management class. It is a singleton, do not instantiate this class manually, it should be accessed through `Laya.timer`.

At the same time, `Laya.Timer` represents the main clock of the game, and is also the clock for managing scenes, animations, easing and other effects. By controlling the scaling of this clock, the effect of fast forward and slow playback can be achieved.

## 1. Frame interval

### 1.1 Execute once regularly (based on frame rate)

`Laya.timer.frameOnce` is defined as follows:

```typescript
	/**
 	* Executed once at a scheduled time (based on frame rate).
 	* @param delay How many frames to delay (unit is frame).
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	* @param args callback parameters.
 	* @param coverBefore Whether to cover the previous delayed execution, the default is true.
 	*/
	frameOnce(delay: number, caller: any, method: Function, args: any[] = null, coverBefore: boolean = true): void {
    	this._create(true, false, delay, caller, method, args, coverBefore);
	}
```

Usage examples are as follows:

```typescript
const { regClass } = Laya;
import { RuntimeScriptBase } from "./RuntimeScript.generated";

@regClass()
export class RuntimeScript extends RuntimeScriptBase {
	onAwake(): void {
    	//After 60 frames, the transparency of the image becomes 0.5
    	Laya.timer.frameOnce(60, this, () => {
        	this.Image.alpha = 0.5;
    	})
	}
}
```

### 1.2 Timed repeated execution (based on frame rate)

`Laya.timer.frameLoop`, defined as follows:

```typescript
	/**
 	* Scheduled repeat execution (based on frame rate).
 	* @param delay The number of frames (unit: frame).
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	* @param args callback parameters.
 	* @param coverBefore Whether to cover the previous delayed execution, the default is true.
 	*/
	frameLoop(delay: number, caller: any, method: Function, args: any[] = null, coverBefore: boolean = true): void {
    	this._create(true, true, delay, caller, method, args, coverBefore);
	}
```

Usage examples are as follows:

```typescript
const { regClass } = Laya;
import { RuntimeScriptBase } from "./RuntimeScript.generated";

@regClass()
export class RuntimeScript extends RuntimeScriptBase {
	onAwake(): void {
    	//After every 60 frames, the transparency of the image is reduced by 0.1
    	Laya.timer.frameLoop(60, this, () => {
        	this.Image.alpha -= 0.1;
    	})
	}
}
```



## 2. Time interval

### 2.1 Execute once regularly (unit: milliseconds)

`Laya.timer.once`, defined as follows:

```typescript
	/**
 	* Execute once regularly.
 	* @param delay delay time (unit: milliseconds).
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	* @param args callback parameters.
 	* @param coverBefore Whether to cover the previous delayed execution, the default is true.
 	*/
	once(delay: number, caller: any, method: Function, args: any[] = null, coverBefore: boolean = true): void {
    	this._create(false, false, delay, caller, method, args, coverBefore);
	}
```

Usage examples are as follows:

```typescript
const { regClass } = Laya;
import { RuntimeScriptBase } from "./RuntimeScript.generated";

@regClass()
export class RuntimeScript extends RuntimeScriptBase {
	onAwake(): void {
    	//After 1 second, the transparency of the image becomes 0.5
    	Laya.timer.once(1000, this, () => {
        	this.Image.alpha = 0.5;
    	})
	}
}
```

### 2.2 Scheduled repeated execution (unit: milliseconds)

`Laya.timer.loop` is defined as follows:

```typescript
	/**
 	*Repeated execution at regular intervals.
 	* @param delay interval time (unit milliseconds).
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	* @param args callback parameters.
 	* @param coverBefore Whether to cover the previous delayed execution, the default is true.
 	* @param jumpFrame Whether the clock jumps frame. Time-based cyclic callback. If multiple callbacks can be executed within a unit time interval, for performance reasons, the engine will only execute it once by default. After setting jumpFrame=true, the callback will be executed multiple times continuously.
 	*/
	loop(delay: number, caller: any, method: Function, args: any[] = null, coverBefore: boolean = true, jumpFrame: boolean = false): void {
    	var handler: TimerHandler = this._create(false, true, delay, caller, method, args, coverBefore);
    	if (handler) handler.jumpFrame = jumpFrame;
	}
```

Usage examples are as follows:

```typescript
const { regClass } = Laya;
import { RuntimeScriptBase } from "./RuntimeScript.generated";

@regClass()
export class RuntimeScript extends RuntimeScriptBase {
	onAwake(): void {
    	//After every 1 second, the transparency of the image is reduced by 0.1
    	Laya.timer.loop(1000, this, () => {
        	this.Image.alpha -= 0.1;
    	})
	}
}
```



## 3. Pause timer execution

Once the timer is paused, the game will be at rest:

```typescript
	/**
 	* Pause the clock
 	*/
	pause(): void {
    	this.scale = 0;
	}

	/**
 	*Restore clock
 	*/
	resume(): void {
    	this.scale = 1;
	}
```



## 4. Delayed execution of current frame

Executed immediately after the current frame is executed. Executed before rendering, the execution priority is higher than the timer delayed by one frame:

```typescript
	/**
 	* Delayed execution.
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	* @param args callback parameters.
 	*/
	callLater(caller: any, method: Function, args: any[] = null): void {
    	CallLater.I.callLater(caller, method, args);
	}
```

Usage examples are as follows:

```typescript
const { regClass } = Laya;
import { RuntimeScriptBase } from "./RuntimeScript.generated";

@regClass()
export class RuntimeScript extends RuntimeScriptBase {
	onAwake(): void {
    	//The loop is called 10 times, but the timer callback function is only executed once, that is, the "hideImage" log is only printed once
    	for (let i = 0; i < 10; i++)
        	Laya.timer.callLater(this, this.hideImage);
	}

	hideImage(): void {
    	console.log("hideImage");
    	this.Image.visible = false;
	}
}
```



## 5. Clean up timer

`Laya.timer.clear`: Clear the specified timer. The definition is as follows:

```typescript
	/**
 	* Cleanup timer.
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	*/
	clear(caller: any, method: Function): void {
    	var handler: TimerHandler = this._getHandler(caller, method);
    	if (handler) {
        	handler.clear();
    	}
	}
```

`Laya.timer.clearAll`: Clear all timers in the specified scope of the object. The definition is as follows:

```typescript
	/**
 	* Clear all timers on the object.
 	* @param caller execution domain (this).
 	*/
	clearAll(caller: any): void {
    	if (!caller) return;
    	for (var i: number = 0, n: number = this._handlers.length; i < n; i++) {
        	var handler: TimerHandler = this._handlers[i];
        	if (handler.caller === caller) {
            	handler.clear();
        	}
    	}
	}
```

It is recommended to clear the timer or clear all timers before a module function is destroyed.



## 6. Execute and delete the timer immediately

`Laya.timer.runCallLater`: Execute callLater immediately and delete it after execution. The definition is as follows:

```typescript
	/**
 	* Execute callLater immediately.
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	*/
	runCallLater(caller: any, method: Function): void {
    	CallLater.I.runCallLater(caller, method);
	}
```

Usage examples are as follows:

```typescript
const { regClass } = Laya;
import { RuntimeScriptBase } from "./RuntimeScript.generated";

@regClass()
export class RuntimeScript extends RuntimeScriptBase {
	onAwake(): void {
    	//Delay one frame to execute hideImage
    	Laya.timer.callLater(this, this.hideImage);
    	//The current frame executes hideImage immediately and deletes the timer after execution.
    	Laya.timer.runCallLater(this, this.hideImage);
	}

	hideImage(): void {
    	console.log("hideImage");
    	this.Image.visible = false;
	}
}
```

> Note: The above example will print a "hideImage" because runCallLater in LayaAir will delete the timer after executing the callback. If the timer is not deleted, after the current frame executes the callback hideImage of runCallLater, the callback hideImage of callLater will also be executed, and two "hideImages" will be printed.

`Laya.timer.runTimer`: Execute the timer immediately in advance and delete it from the queue after execution. The definition is as follows:

```typescript
	/**
 	* Immediately execute the timer in advance and delete it from the queue after execution
 	* @param caller execution domain (this).
 	* @param method timer callback function.
 	*/
	runTimer(caller: any, method: Function): void {
    	var handler: TimerHandler = this._getHandler(caller, method);
    	if (handler && handler.method != null) {
        	this._map[handler.key] = null;
        	handler.run(true);
    	}
	}
```

Usage examples are as follows:

```typescript
const { regClass } = Laya;
import { RuntimeScriptBase } from "./RuntimeScript.generated";

@regClass()
export class RuntimeScript extends RuntimeScriptBase {
	onAwake(): void {
    	//If there is no Laya.timer.runTimer, hideImage will be executed after 5 seconds.
    	Laya.timer.loop(5000, this, this.hideImage);
    	//Execute hideImage immediately
    	Laya.timer.runTimer(this, this.hideImage);
	}

	hideImage(): void {
    	console.log("hideImage");
    	this.Image.visible = false;
	}
}
```




