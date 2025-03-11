# Detailed explanation of resource loading method of LayaAir engine

> Author: Charley   

Resource loading is the core module of the game engine and a must-use function. The LayaAir engine provides the Loader class for loading text, JSON, binary, images and other resources.

Next, we will thoroughly master the use of various resource loading in this document.

## 1. Load a single resource

### 1.1 How to load common resources

Usually, the `Laya.loader.load("resource path")` method is used to load a single resource, and `.then("callback method")` is used to handle the post-loading logic.

```typescript
Laya.loader.load(url).then((res)=> {/** Without type, used for regular resources */});
```

The complete script example is as follows:

```typescript
const { regClass, property } = Laya;

@regClass()
export class LoaderDemo extends Laya.Script {

	onAwake(): void {
    	this.loadTexture("resources/image/monkey2.png", 500);//You need to put the corresponding resources in resources/image
    	this.loadTexture("https://layaair.com/3.x/demo/resources/res/apes/monkey2.png");
	}

	/**Load and display images */
	loadTexture(url: string, x: number = 0, y: number = 0): void {
    	Laya.loader.load(url).then((res: Laya.Texture) => {
        	let img = new Laya.Image();
        	img.texture = res;
        	// img.skin = url; //UI components can also set skin directly
        	img.pos(x, y);
        	this.owner.addChild(img);
    	});
	}
}
```

### 1.2 Typed loading method

Sometimes, network resources do not have a suffix, or the picture xxx.png is not used as a Texture and needs to be defined as a TextureCube. Therefore, these times need to be distinguished by type.

The usage is basically the same as without type, except that the second parameter of load is added: the type identification string.

```typescript
Laya.loader.load(url, type).then((res)=> {
	//With type, used to distinguish resources without suffix, or resources with different functions of the same suffix.
	//For example, the picture xxx.png is defined as TextureCube, use load("xxx.png", Laya.Loader.TEXTURECUBE). What you get is a TextureCube.
});
```

The complete script example is as follows:

```typescript
const { regClass, property } = Laya;

@regClass()
export class LoaderDemo extends Laya.Script {

	onAwake(): void {
    	//The following URL may become invalid in the future and is only used for example reference.
    	this.loadHTMLImage("https://layaair.com/3.x/demo/resources/res/apes/monkey2.png");
	}

	/** Load URL without suffix resources */
	loadHTMLImage(url: string): void {
    	Laya.loader.load(url, Laya.Loader.IMAGE).then((res: Laya.Texture) => {
        	let sp = new Laya.Sprite();
        	sp.texture = res;
        	this.owner.addChild(sp);
    	});
	}
}
```

**Commonly used types are as follows:**

| Engine global variables	| Type identification string | Type description	|
| ----------------------- | -------------- | ------------------------------------------------------------ |
| Laya.Loader.TEXT    	| text       	| Text type	|
| Laya.Loader.JSON    	| json       	| JSON type	|
| Laya.Loader.XML     	| xml        	| XML type	|
| Laya.Loader.BUFFER  	| arraybuffer	| Binary type	|
| Laya.Loader.IMAGE   	| image      	| Texture type	|
| Laya.Loader.SOUND   	| sound      	| sound type	|
| Laya.Loader.VIDEO   	| video      	| Video type	|
| Laya.Loader.ATLAS   	| atlas      	| Atlas type, return the json information of the album after loading is completed (and create a small image Texture in the album) |
| Laya.Loader.HIERARCHY   | HIERARCHY  	| Hierarchical resources, such as scene ls files and prefab lh files	|
| Laya.Loader.FONT    	| font       	| Bitmap font type	|
| Laya.Loader.TTF     	| ttf        	| TTF font type	|
| Laya.Loader.MESH    	| MESH       	| Mesh resources	|
| Laya.Loader.MATERIAL	| MATERIAL   	| Material resources	|
| Laya.Loader.TEXTURE2D   | TEXTURE2D  	| Texture2D resources	|
| Laya.Loader.TEXTURECUBE | TEXTURE2D  	| TextureCube Resources	|
| Laya.Loader.SPINE   	| SPINE      	| Spine Resources	|



## 2. Load multiple resources

Many times, we need to preload a large amount of resources on the Loading interface. In this way, after entering the game after passing the loading progress bar, the resources will be displayed directly, and there will be no waiting for the resources to be loaded.

At this time, we need to use multiple resource loading methods.

### 2.1 Multi-resource loading method that omits types

If we only load the more commonly used resources, which can be identified by the file suffix, then it is definitely more concise to omit the type.

Loading multiple APIs is basically the same as loading a single one, except that when loading multiple ones, the first parameter is an array of resource addresses.

```typescript
Laya.loader.load([url1, url2]).then((res:Array<any>)=> { /** Load multiple, without type, used for regular resources */ });	 
```

The complete script example is as follows:

```typescript
const { regClass, property } = Laya;

@regClass()
export class LoaderDemo extends Laya.Script {

	onAwake(): void {
    	let resArr: Array<string> = [
        	"image/bird.jpg",
        	"image/cloud.png",
        	"https://layaair.com/3.x/demo/resources/res/apes/monkey2.png"
    	];
    	this.loadTextures(resArr);
	}

	/** Load multiple resources
 	* @param url resource array
	*/
	loadTextures(url: Array<string>): void {
    	Laya.loader.load(url).then((res: Array<Laya.Texture>) => {
        	let sp: Laya.Sprite = new Laya.Sprite();
        	this.owner.addChild(sp);
        	let i: number = 0;
        	//Redraw every second
        	Laya.timer.loop(1000, this, () => {
            	i == res.length && (i = 0);
            	//Clear all drawings of sp nodes (excluding child nodes)
            	sp.graphics.clear();
            	//Redraw the texture on the sp node
            	sp.graphics.drawTexture(res[i]);
            	i++;
        	});
    	});
	}
}
```

### 2.2 Typed multi-resource loading method

If most resources do not need to be typed, and some resources need to be distinguished by type, how to use the loading method?

In the resource array, the type needs to be passed in, and the type that is not required can still be omitted.

```typescript
Laya.loader.load([url1, { url:url2, type: type }]).then((res:Array<any>)=> { /** Load multiple, set the type as needed to meet various needs */ });	 
```

The complete script example is as follows:

```typescript
const { regClass, property } = Laya;

@regClass()
export class LoaderDemo extends Laya.Script {

	onAwake(): void {
    	let resArr: Array<any> = [
        	"image/bird.jpg",
        	"image/cloud.png",
        	{
            	url: "https://layaair.com/3.x/demo/resources/res/apes/monkey2.png",
            	type: "image"
        	},
        	{
            	url: "https://layaair.com/3.x/demo/resources/res/apes/monkey1.png",
            	type: Laya.Loader.IMAGE
        	}
    	];
    	this.loadTextures(resArr);
	}

	/** Load multiple resources
 	* @param url resource array
	*/
	loadTextures(url: Array<string>): void {
    	//The type returned by the loading callback, if it is diverse, it is best to be any
    	Laya.loader.load(url).then((res: Array<any>) => {
        	let sp: Laya.Sprite = new Laya.Sprite();
        	this.owner.addChild(sp);
        	let i: number = 0;
        	//Redraw every second
        	Laya.timer.loop(1000, this, () => {
            	i == res.length && (i = 0);
            	//Clear all drawings of sp nodes (excluding child nodes)
            	sp.graphics.clear();
            	//Redraw the texture on the sp node
            	sp.graphics.drawTexture(res[i]);
            	i++;
        	});
    	});
	}
}
```

> The type in the example deliberately uses two methods: string identification and engine global variable identification. Developers can use either one. For details, refer to the type identification table in 1.2.

### 2.3 Unified multi-resource loading method

Sometimes, multiple resources loaded need to use type identifiers, and these types are all of the same type. For example, all resources loaded are URL resources without suffixes.

At this time, we can set the type identifier uniformly.

```typescript
Laya.loader.load([url1, url2], type).then((res:Array<any>)=> { /** Load multiple, unified type, simple and practical */ });	 
```

The complete script example is as follows:

```typescript
const { regClass, property } = Laya;

@regClass()
export class LoaderDemo extends Laya.Script {

	onAwake(): void {
    	let resArr: Array<any> = [
        	"image/bird.jpg",
        	"https://layaair.com/3.x/demo/resources/res/apes/monkey2.png",
        	"image/cloud.png",
        	"https://layaair.com/3.x/demo/resources/res/apes/monkey2.png",
    	];
    	this.loadTextures(resArr);
	}

	/** Load multiple resources
 	* @param url resource array
	*/
	loadTextures(url: Array<string>): void {
    	//Load the type returned by the callback, if not sure, just any
    	Laya.loader.load(url, Laya.Loader.IMAGE).then((res: Array<Laya.Texture>) => {
        	let sp: Laya.Sprite = new Laya.Sprite();
        	this.owner.addChild(sp);
        	let i: number = 0;
        	//Redraw every second
        	Laya.timer.loop(1000, this, () => {
            	i == res.length && (i = 0);
            	//Clear all drawings of sp nodes (excluding child nodes)
            	sp.graphics.clear();
            	//Redraw the texture on the sp node
            	sp.graphics.drawTexture(res[i]);
            	i++;
        	});
    	});
	}
}
    
```



## 3. Do not parse or cache loaded resources

The `load()` method provided by the engine loads resources and may do some encapsulation after parsing.

For example, if we use the `load()` method to load a json data, we need to be in data to get the data in the json file.

Use the `load()` method, the example is as follows:

```typescript
onEnable(): void {
	const jsonPath: string = "json/bagList.json";
	Laya.loader.load(jsonPath).then((json) => {
    	let _json = json.data;
    	if (_json.bagList && _json.bagList.length > 0) {
        	//Pass json data to the data source attribute array of the list component
        	this.bagList.array = _json.bagList;
    	}
	});
}
```

And when we use the `fetch()` method, we can directly get the data in the json file. However, it should be noted that resources loaded using fetch will not be cached and can only be used in the callback after the loading is completed. The loading cache cannot be read through getRes.

Use the `fetch()` method, the example is as follows:

```typescript
onEnable(): void {
	const jsonPath: string = "json/bagList.json";
	//fetch gets the original data, but there is no cache and cannot be obtained with getRes.
	Laya.loader.fetch(jsonPath, "json").then((_json) => {  
    	if (_json.bagList && _json.bagList.length > 0) {
        	//Pass json data to the data source attribute array of the list component
        	this.bagList.array = _json.bagList;
    	}
	});
}
```

**fetch loading is limited to the following types**:

| type identification string | resource type	|
| -------------- | ----------------------------- |
| text       	| string                    	|
| json       	| any                       	|
| xml        	| XMLDocument               	|
| arraybuffer	| ArrayBuffer               	|
| image      	| HTMLImageElement \ImageBitmap |
| sound      	| HTMLAudioElement          	|

When using it, just fill in the corresponding type identification string in the second parameter.

For more fetch loading code examples, please refer to the [*Using Binary Images*](../../../2D/useBinaryImage/readme.md) document

> **Tips**​
>
> Unless the purpose of fetch is clearly understood, it is not recommended for developers to use it.



## 4. Option parameters

When using the `load()` method or the `fetch()` method to load resources, the Option parameter can be used, such as code:

```typescript
//Create Option
let option:any = {};
option.blob = this.imgBlob;
//Get HTMLImageElement by passing blob object
Laya.loader.fetch("" ,"image", null, option).then((res)=>{
});
```

Specify the `fetch()` method through the Option parameter and pass the blob object to obtain HTMLImageElement

Currently supported Option parameters:

```typescript
export interface ILoadOptions {
	type?: string; //Resource type. For example: Loader.IMAGE.
	priority?: number; //(default = 0) The priority of loading. The larger the number, the higher the priority. Those with higher priority are loaded first.
	group?: string; //Group to facilitate resource management.
	cache?: boolean; // Whether to cache
	noRetry?: boolean; //Whether to retry loading
	silent?: boolean; //Whether to prompt loading failure
	useWorkerLoader?: boolean; //(default = false) Whether to use worker loading (only for IMAGE type and ATLAS type, and effective when the browser supports it)
	constructParams?: TextureConstructParams; //Picture properties, reference is as follows
	propertyParams?: TexturePropertyParams; //Texture properties, reference is as follows
	blob?: ArrayBuffer; //Pass the blob object to get HTMLImageElement
	noMetaFile?: boolean; //Do not download Meta(json) files
	[key: string]: any;
}
    
TextureConstructParams {
	width?: number,
	height?: number,
	format?: TextureFormat,
	mipmap?: boolean,
	canRead?: boolean,
	sRGB?: boolean,
}
    
TexturePropertyParams {
	wrapModeU?: number,
	wrapModeV?: number,
	filterMode?: FilterMode,
	anisoLevel?: number,
	premultiplyAlpha?: boolean,
	hdrEncodeFormat?: HDREncodeFormat,
}
```






