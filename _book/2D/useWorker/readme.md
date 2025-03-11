# Multi-threaded worker



> The workers in this document are only supported in HTML5 mode in browser mode. Workers are not currently supported in the LayaNative packaged APP solution.

Browsers are single-threaded in the traditional sense that they force all scripts in an application to run together in a single UI thread. Although you can create the illusion that multiple tasks are running simultaneously by using Document Object Model (DOM) events and APIs such as setTimeout, it only takes one computationally intensive task to drastically degrade the user experience. The worker function was introduced in html5. By using Web Worker, we can run JavaScript in the background of the browser without occupying the browser's own thread. Web Workers can improve the overall performance of the application and improve the user experience. Threads can perform tasks without interfering with the user interface.



### Native worker

Web workers are divided into two types, dedicated thread dedicated web worker, and shared thread shared web worker. The Dedicated web worker ends when the current page is closed; this means that the Dedicated web worker can only be accessed by the page that created it. The corresponding Shared web worker can be accessed by multiple pages. However, web workers have some limitations, and not all interfaces and methods can be used.

- Web Worker cannot access DOM nodes;

- Web Worker cannot access global variables or global functions;

- Web Worker cannot call functions such as alert() or confirm;

- Web Worker cannot access browser global variables such as window and document;

  The [Functions supported by worker](https://developer.mozilla.org/En/DOM/Worker/Functions_available_to_workers) page provides a list of global functions supported by the worker. Developers can look at the corresponding methods themselves.

#### Method Overview

##### Constructor Worker()

This constructor creates a web worker that can execute a script located at the specified URL. Scripts must comply with the [Same Origin Policy](https://developer.mozilla.org/en/Same_origin_policy_for_JavaScript).

##### postMessage()：

Deliver messages to the worker's inner scope. This method receives a single parameter, the data to be passed to the worker. The data can be any value or processed by the [structured copy](http://www.whatwg.org/specs/web-apps/current-work/multipage/common-dom-interfaces.html#transferable) algorithm JavaScript objects, in other words, can contain circular references.

###### #Parameters

- aMessage

  The object passed to the worker; it will be contained in the data field of the event object passed to the onmessage handler. You can pass any value or one processed by the [structured copy](http://www.whatwg.org/specs/web-apps/current-work/multipage/common-dom-interfaces.html#transferable) algorithm JavaScript objects, which can contain circular references.

- transferList

  An optional array of objects to transfer ownership of. If ownership of an object is transferred, it will no longer be available within the original context, but will only be available within the worker to which it was transferred.

##### terminate()

Terminate the worker immediately. This method does not leave any opportunity for the worker to complete the operation; it simply stops immediately

### Attributes

| Property	| Type                                                     	| Description                                              	|
| :---------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `onmessage` | [`EventListener`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventListener) | An event listening function, whenever a MessageEvent with `message attribute` is sent from the worker This function will be executed when it bubbles out. The `data` attribute of the event stores the message content. |
| `onerror` | [`EventListener`](https://developer.mozilla.org/zh-CN/docs/Web/API/EventListener) | An event listening function, whenever an `ErrorEvent of type `error` comes from This function will be executed when it bubbles up in the worker. ` |

Next we use native js to see how to use it.

Create a new js file and put it in index.html. code show as below:

```javascript
var myWorker = new Worker("js/my_task.js");
myWorker.onmessage = function (oEvent) {
    console.log("Called back by the worker!\n");
};
myWorker.postMessage("start"); // start the worker.
```

Create a new my_task.js file and place it in the js folder. The code is as follows:

```javascript
self.addEventListener('message', function (e) {
	var xmlreq = new XMLHttpRequest();
	xmlreq.responseType = "text";
	xmlreq.onload = function (e) {
    	var data = e.currentTarget.response;
    	self.postMessage(data);
	}
	xmlreq.open("get","../atlas/comp.json");
	xmlreq.send()
}, false);
```

This example loads the file in the worker and passes it to the main process after loading. You can see the data output in the browser console by running this example.

`var myWorker = new Worker("my_task.js")` instantiates a worker, passes in a js file, and notifies the worker thread to start through `myWorker.postMessage("start")`.

`self.addEventListener('message',xxx)`Listens for messages notified by the main thread.

`self.postMessage(data);`Send data to the main thread.

Note: Web worker does not support the file protocol, so it cannot be run when opened directly. Developers can cooperate with the IDE's built-in server and run it through the URL to see the effect. Open the console and you can see that the data has been printed.

### Application in Laya

Workers are encapsulated internally in LayaAir 3.0 to solve the problem of stuck loading decoded images. Developers can turn on the switch or customize workers to solve the problem of CPU consumption in the project. We will introduce them separately below.

Create a new project. For convenience of display, we create a new ui project. The simple calling interface is as follows:

```typescript
class LayaUISample {
	constructor() {
    	//Initialize the engine
    	Laya.init(600,400,Laya.WebGL);
    	//Set the worker.js path provided by Laya
    	Laya.WorkerLoader.workerPath = "libs/worker.js";
    	//Start the worker thread
    	Laya.WorkerLoader.enable = true;
    	//Load the resources needed by the engine
    	Laya.loader.load("../atlas/comp.atlas",Laya.Handler.create(this,this.onLoaded));
	}
	private onLoaded():void{
    	//Instance UI interface
    	var testView:ui.test.TestPageUI = new ui.test.TestPageUI();
    	Laya.stage.addChild(testView);
	}
}
new LayaUISample;
```

`WorkerLoader.workerPath = "libs/worker.js";`

Set the path of worker.js. This worker.js is officially provided by Laya. We copy it to the path we set ourselves. This js is in Laya's engine library. What I set here is under libs.

`WorkerLoader.enable = true;`

Turning on the worker mode to load decoded images greatly relieves the pressure of decoding on the main thread.

The above method is the official decoding method of 2.0. In LayaAir3.0, we can also customize workers to optimize the CPU-consuming areas of the project.

The following uses a simple example to demonstrate usage. We can transplant the js script at the beginning of the tutorial.

```typescript
var worker:any = Laya.Browser.window.Worker("js/my_task.js");
worker.onmessage = function(oEvent):void{
	console.log("Called back by the worker!\n");
};
worker.postMessage("start"); // start the worker.
```

The code in my_task.js still loads a file. code show as below:

```typescript
self.addEventListener('message', function (e) {
	var xmlreq = new XMLHttpRequest();
	xmlreq.responseType = "text";
	xmlreq.onload = function (e) {
    	var data = e.currentTarget.response;
    	self.postMessage(data);
	}
	xmlreq.open("get","../atlas/comp.atlas");
	xmlreq.send()
}, false);
```

Compile and run the code, and you can see that the console outputs the data we loaded comp.atlas.

Summary: Web workers are generally used to parse and load large files, such as large json files. Time-consuming calculations, or some resources that do not need to be loaded immediately can be completed in the background thread, so that users basically cannot feel the main thread. lag. Enhance project fluidity. Improve user experience.

- For details on `Web Workers`, please see [W3C's xhr standard](https://www.w3.org/TR/workers/);
- For detailed API and introduction, please refer to [here](https://developer.mozilla.org/en-US/docs/Web/API/Worker/)
- The [Functions supported by worker](https://developer.mozilla.org/En/DOM/Worker/Functions_available_to_workers) page provides a list of global functions supported by the worker.

