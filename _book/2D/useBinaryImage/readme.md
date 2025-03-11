# Use binary images



## 1. Pictures and Binary

In the era of web games, in order to prevent resources from being stolen, a common practice is to encrypt resources such as pictures. The so-called encryption is to disrupt the original storage bytes of the resource, or intersperse some things. But in the HTML5 era, it was found that most images were loaded directly. Why is the approach different from the web game era? Is it because html5 cannot load decoded binary images? of course not. The reason why encryption is not performed is mainly because the source code of our project is completely exposed on the browser side. There is no secret at all. Even if it is encrypted, you can get your source code by writing a script and executing it. But in order to meet the needs of developers in this regard, let us explain how LayaAir3.0 performs binary image operations.



## 2. How to load

Regarding how to load, here we start with the native one and then transition to the LayaAir engine so that developers can understand the meaning. Loading in binary stream, here we use XMLHttpRequest binary stream to load. We will not describe the operation of XMLHttpRequest here. Let's try loading it in binary mode first. Here we first use js script to operate. code show as below:

```javascript
var xhr = new XMLHttpRequest();
xhr.open("get", "res/atlas/comp.png", true);
xhr.responseType = "arraybuffer";
xhr.onload = function () {
	if (this.status == 200) {
    	var blob = new Blob([this.response], { type: "image/png" });
    	var img = document.createElement("img");
    	img.onload = function (e) {
        	window.URL.revokeObjectURL(img.src); // Clear release;
    	};
    	img.src = window.URL.createObjectURL(blob);
    	document.body.appendChild(img);
	}
}
xhr.send();
```

The above method uses the method provided by the browser itself to convert binary into images. There are actually many ways to convert binary into images, such as loading in the binary, decoding it into base64, and then assigning it to you img, or converting the binary data Use canvas to draw the picture, and then assign toDataURL to the src of your img, etc. There are many methods. Here we will use the simplest and most effective method to convert the picture.

After the image is loaded, instantiate an XMLHttpRequest object xhr, set the `responseType` attribute to `arraybuffer`, instantiate a Blob object `blob`, used to create an img tag, `window.URL.createObjectURL(blob)` creates a Point to the URL of the parameter object, and add the created img object to the body of the web page for display. Embed this code into the index.html file and run it to see that the web page displays our images normally.



## 3. How to use it in Laya

We wrote the above simple example using js script, so how to use it in the LayaAir3.0 project?

Add the following code to the script in the project:

```typescript
//test.bin is a binary image. The encrypted image data is four bytes of data written in front of the image.
Laya.loader.fetch("resources/res/test.bin","arraybuffer").then((res)=>{

	//Get the ArrayBuffer data of res
	let arraybuffer: ArrayBuffer = res;
	//Byte array receives arraybuffer
	let byte:Byte = new Byte(arraybuffer);
	//Read data starting from the fourth byte
	byte.writeArrayBuffer(arraybuffer,4);
	//Get the final ArrayBuffer
	let imageArrayBuffer = byte.buffer;
	//Instantiate a Blob object blob to create an img tag
	let imgBlob = new Blob([imageArrayBuffer], { type: "image/png" });
    
	//Convert to Base64 image format
	let reader = new FileReader();
	reader.readAsDataURL(imgBlob);
	reader.onload = (e)=> {	 
   	 
   	 let sp1:Sprite = new Sprite();
    	//Load Base64 image data
    	sp1.loadImage(e.target.result as string);	 
    	this.owner.addChild(sp1);
	}      

});    
```

In the above code, Laya.loader.fetch is used to load image binary data. According to custom rules, the data encryption method can be parsed and the complete image data can be obtained. Here we introduce more about the Laya.loader.fetch method of the LayaAir3.0 engine. The advantage of using Laya.loader.fetch is that it is a relatively low-level method for downloading resources. Unlike the load method, it does not parse the returned data and does not cache the downloaded content.

After selecting the ArrayBuffer of image data, you can create a Blob object of Image, which can be converted into Base64 image data to display the image through the FileReader class. The image is not displayed here using DOM, but drawn through Laya.Sprite.

Developers can also use Laya.Texture to render through Laya.Sprite's drawTexture method. The code is as follows:

```typescript
//Create a url object;
var url:string = Laya.Browser.window.URL.createObjectURL(imgBlob);
//Load URL to get HTMLImageElement
Laya.loader.fetch( url,"image" ).then((res)=>{

	//Create Texture2D
	var t2d: Texture2D = new Texture2D(res.width, res.height, TextureFormat.R8G8B8A8, false, false, true);
	t2d.setImageData(res, true, false);

	//Create Texture
	var texture: Texture = new Texture(t2d);

	let sp2:Sprite = new Sprite();
	//Use the texture drawing method of Sprite object
	sp2.graphics.drawTexture(texture, 150, 0);
	this.owner.addChild(sp2);

});
```

After the Blob object is created, create a URL pointing to the parameter object through `window.URL.createObjectURL(blob)`, and then load the URL through Laya.loader.fetch to obtain the HTMLImageElement object. The HTMLImageElement object can be obtained through setImageData of Laya.Texture2D. The data is converted to Laya.Texture2D, and finally Laya.Texture is created to draw

In the above code, you can also use Laya.loader.fetch by passing the Option parameter. You can pass the blob object directly as a parameter. The code is as follows:

```typescript
//Create Option
let option:any = {};
option.blob = imgBlob;
//Get the HTMLImageElement object by passing the Option parameter, which contains the blob object.
Laya.loader.fetch( "" ,"image", null, option).then((res)=>{

	//Create Texture2D
	var t2d: Texture2D = new Texture2D(res.width, res.height, TextureFormat.R8G8B8A8, false, false, true);
	t2d.setImageData(res, true, false);

	//Create Texture
	var texture: Texture = new Texture(t2d);

	let sp2:Sprite = new Sprite();
	//Use the texture drawing method of Sprite object
	sp2.graphics.drawTexture(texture, 150, 0);
	this.owner.addChild(sp2);
});
```

The above method is the processing method of binary images. Developers can formulate more binary data rules according to needs, and can package many images into a picture collection file, parse and load at one time.






