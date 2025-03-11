# Telecommunication



## 1. Overview

During our project development process, apart from the fact that a single machine does not need to use network communication, when developing a network project, it is inevitable to deal with network communication. This chapter will explain the network communication part of LayaAir. Usually we use Http and WebSocket, two network communication methods. First, let’s compare the differences between the two:

- **HTTP**：

Advantages: The protocol is mature, widely used, based on TCP/IP, has the reliability of TCP, low R&D cost, rapid development, and is widely supported, such as nginx/apache/tomcat, etc.

Disadvantages: stateless and connectionless, only PULL mode, PUSH is not supported, data packets are large, and all header information must be included in each request.

Features: stateless, connectionless (short connection), supports C/S mode, suitable for text transmission, but can also transmit other types of data, such as pictures, videos, etc.

- **WebSocket**：

Advantages: The protocol is relatively new, but has become mature. It is based on TCP/IP and has the reliability of TCP. The data message is smaller. Only the handshake phase when establishing a connection requires larger header information, and the header information in the subsequent data exchange phase is The information is small, connection-oriented, stateful protocol, and supports the server to actively push data (PUSH mode).

Disadvantages: WebSocket is an application layer protocol. Although the data packets are relatively concise, compared with the TCP/IP protocol, its data packet header information is relatively large. For very small data packets, this may cause some additional traffic consumption. In addition, more server resources may be consumed due to the need to maintain the connection.

Features: Stateful, connection-oriented, small data header, supports full-duplex communication, suitable for applications requiring real-time communication.



Through the above analysis of protocol characteristics, it is recommended that:

1. For weak network games, such as elimination games and card games, you can use HTTP protocol directly. If security is considered, use HTTPS directly, or use symmetric encryption on the content body;

2. For real-time and interactive requirements, and the team has relevant experience, you can give priority to the websocket protocol, such as large online games such as SLG and RPG.



## 2. Http connection

The HTTP protocol, the Hypertext Transfer Protocol, is the basis of Web networking and one of the protocols commonly used in mobile phone networking. The HTTP protocol is an application built on the TCP protocol.



### 2.1 Laya.HttpRequest

In the LayaAir engine, `HttpRequest` is the basic class for us to send requests. The `HttpRequest` class actually wraps the native `XMLHttpRequest`. Let’s first understand `HttpRequest`.

#### 2.1.1 Native XMLHttpRequest object

```typescript
	/**
 	* The native XMLHttpRequest reference encapsulated by this object.
 	*/
	get http(): any {
    	return this._http;
	}
```

`XMLHttpRequest` is available through the ._http attribute. `XMLHttpRequest` in Chinese can be interpreted as extensible hypertext transfer request. It provides the client with the functionality to transfer data between the client and the server. It provides a simple way to get data via a URL without causing the entire page to refresh. This allows the web page to update only part of the page without disturbing the user.

1. Properties

| Properties	| Type	| Description	|
| :------------------- | :--------------------------- | ------------------------------------------------------------ |
| `onreadystatechange` | `function`               	| A JavaScript function object that will be called when the readyState property changes.	|
| `readyState`     	| `unsigned short`         	| Five statuses of requests	|
| `response`       	| `varies`                 	| The type of response entity is specified by responseType, which can be `ArrayBuffer`, `Blob`, [`Document`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document), JavaScript Object (i.e. "json"), or a string. If the request is incomplete or failed, the value is `null` |
| `responseText`   	| `DOMString`              	| The response to this request is text, or `null` when the request was unsuccessful or has not yet been sent. Read-only. |
| `responseType`   	| `XMLHttpRequestResponseType` | Setting this value changes the response type. Just tell the server the response format you expect. |
| `status`         	| `unsigned short`         	| The response status code for the request (for example, status code 200 indicates a successful request). Read-only |
| `statusText`     	| `DOMString`              	| Response status information for this request, consisting of a status code and reason phrase (such as "`200 OK`"). Read only |
| `upload`         	| `XMLHttpRequestUpload`   	| You can add an event listener on upload to track the upload process.	|
| `withCredentials`	| `boolean`                	| Indicates whether to use authentication information (such as cookies or authorization headers) when making cross-site Access-Control requests. Defaults to `false` |
| `timeout`        	| `number`                 	| Request timeout	|

2. Method

`abort()`

If the request has already been sent, the request is aborted immediately.

`getAllResponseHeaders()`

Returns all response header information (response header name and value), or `null` if the response header has not been accepted.

`getResponseHeader()`

Returns the value of the specified response header. If the response header has not been accepted or the response header does not exist, null is returned.

`open()`

Initialize a request.

`send()`

Send the request. If the request is in asynchronous mode (default), this method will return immediately. In contrast, if the request is in synchronous mode, the method will not return until the response to the request has been fully accepted.

`setRequestHeader()`

Assign a value to the specified HTTP request header. Before doing this, you must confirm that you have opened a URL by calling the `open()` method.

Therefore, in the process of using `HttpRequest`, we can also obtain the `XMLHttpRequest` object and perform related operations on the `XMLHttpRequest` object. We will not explain too much about `XMLHttpRequest` here. Developers can check the relevant documents by themselves.

- For detailed information on `XMLHttpRequest`, please see [W3C's xhr standard](https://www.w3.org/TR/XMLHttpRequest/);

- `XMLHttpRequest` sends various types of data, you can refer to [Sending Data](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Sending_and_Receiving_Binary_Data) and [this article on html5rocks ](http://www.html5rocks.com/zh/tutorials/file/xhr2/)

- To understand the basic use of `XMLHttpRequest`, you can refer to [MDN's Introduction to XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest);

- If you want to know about cross-domain requests, you can refer to [W3C's cors standard](https://www.w3.org/TR/cors/);

 

#### 2.1.2 send() method

Send a request. Usually the request is sent asynchronously, and the parameter types of send are as follows:

```typescript
	/**
 	* Send HTTP request.
 	* @param url			 	 The requested address. Most browsers implement a Same Origin Security Policy and require that the URL have the same hostname and port as the text containing the script.
 	* @param data		 		 (default = null)The data sent.
 	* @param    method   		 (default = "get")The HTTP method used for the request. Values ​​include "get", "post", "head".
 	* @param responseType 		 (default = "text") The response type of the web server, which can be set to "text", "json", "xml", "arraybuffer".
 	* @param    headers   		 (default = null) HTTP request header information. The parameters are in the form of a key-value array: key is the name of the header and should not include blanks, colons or newlines; value is the value of the header and should not include newlines. For example ["Content-Type", "application/json"].
 	*/
	send(url: string, data: any = null,
    	method: "get" | "post" | "head" = "get",
    	responseType: "text" | "json" | "xml" | "arraybuffer" = "text",
    	headers: any[] | null = null)
```



#### 2.1.3 Supported event types

The basic ones we commonly use are progress events, completion events, error events, etc.

```typescript
/**
 * Dispatched when the request progress changes.
 * @eventType Event.PROGRESS
 * */
/*[Event(name = "progress", type = "laya.events.Event")]*/
/**
 * Dispatched after the request is completed.
 * @eventType Event.COMPLETE
 * */
/*[Event(name = "complete", type = "laya.events.Event")]*/
/**
 * Dispatched when request error occurs.
 * @eventType Event.ERROR
 * */
/*[Event(name = "error", type = "laya.events.Event")]*/
```



#### 2.1.4 How to use it in code

The `HttpRequest` used in the LayaAir engine inherits the `EventDispatcher`, which has the function of event dispatching. In addition, it has the function of sending requests. Let's write a simple example to see how to use it:

```typescript
class LayaSample {
	constructor() {
    
   	 //Create HttpRequest object
    	let http: Laya.HttpRequest = new Laya.HttpRequest();
    	//Set the timeout
    	http.http.timeout = 10000;
    	//Sent a simple request
    	http.send("resources/data.txt", "", "get", "text");//You need to create a new data.txt file in the resources folder
    	//Set the completion event and add a callback method
    	http.once(Laya.Event.COMPLETE, this, this.completeHandler);
    	//Set error event and add callback method	 
    	http.once(Laya.Event.ERROR, this, this.errorHandler);
    	//Set progress events and add callback methods	 
    	http.on(Laya.Event.PROGRESS, this, this.processHandler);
	}
    
	private processHandler(data:any): void {
    	console.log("processHandler");
	}
    
	private errorHandler(error:any): void {
    	console.log("errorHandler");
	}
    
	private completeHandler(data:any): void {
    	console.log("completeHandler");
	}
}
new LayaSample();
```



### 2.2 GET

In the above example, we sent a simple request using the get method. Used to obtain a remote file in text format. If we dynamically request remote data, it can be changed to the following format:

```typescript
this.hr = new HttpRequest();
this.hr.once(Event.PROGRESS, this, this.onHttpRequestProgress);
this.hr.once(Event.COMPLETE, this, this.onHttpRequestComplete);
this.hr.once(Event.ERROR, this, this.onHttpRequestError);
//Sent a get request with the parameters name=myname and psword=xxx
this.hr.send('http://xkxz.zhonghao.huo.inner.layabox.com/', null, 'get', 'text');
```

The focus here is the send method, which should be distinguished from the send of `XMLHttpRequest`.



### 2.3 POST

The following method of requesting a data using the post method is as follows:

```typescript
this.hr = new HttpRequest();
this.hr.once(Event.PROGRESS, this, this.onHttpRequestProgress);
this.hr.once(Event.COMPLETE, this, this.onHttpRequestComplete);
this.hr.once(Event.ERROR, this, this.onHttpRequestError);
//Sent a post request with the parameters name=myname and psword=xxx
this.hr.send('http://xkxz.zhonghao.huo.inner.layabox.com/', 'name=myname&psword=xxx', 'post', 'text');
```

**Note: There is a difference between GET and POST requests:**

*GET request parameters are passed through the URL, and POST request parameters are included in the request body.*

*GET requests are less secure than POST requests because the parameters are exposed directly in the URL, so GET requests cannot be used to pass sensitive information.*

*The parameters passed in the url of the GET request are limited in length (there is no limit on the length of the URL in the HTTP protocol. The limit is the limit imposed by the specific browser and server. Different browsers have different length limits). POST has no limit on length.*

*GET request parameters will be completely retained in the browser's history, and POST request parameters will not be retained.*

*GET requests are URL encoded (percent encoding), and POST requests support multiple encoding methods.*



### 2.4 Extend HttpRequest

During the development process, `HttpRequest` may not meet our needs, such as uploading files, setting timeouts, operating form data, etc. Extending `HttpRequest` is very simple. You can inherit `HttpRequest` or simply rewrite the `HttpRequest` class yourself. This depends on the needs of the developer. It is recommended to directly inherit `EventDispatcher` when rewriting `HttpRequest`. Rewriting is to repackage the `XMLHttpRequest` class. Here is a simple demonstration of inheritance:

```typescript
 class HttpRequestExtension extends Laya.HttpRequest {
 	constructor() {
     	super();
 	}
 	public send(url:string,data:any=null,method:string="get", responseType:string="text", headers:any=null):void{
     	super.send(url,data,method,responseType,headers);
         	this._http.upload.onprogress= function(e:any):void
         	{
             	//upload progress
         	}
         	this._http.upload.onload= function(e:any):void
         	{
         	}
         	this._http.upload.onerror= function(e:any):void
         	{
         	}
         	this._http.upload.onabort = function(e:any):void
         	{
         	}
 	}
 }
```

The above is a demonstration of uploading files, adding some upload events of `XMLHttpRequest`. The super.send here simply uses the method of the parent class. Developers can not use it and write another one to meet their own needs.



## 3. WebSocket connection

WebSocket is a technology based on the ws protocol, which makes it possible to establish a full-duplex connection. Websocket is commonly used in browsers, but this protocol is not restricted by the platform it is used on.

The format of data sent by websocket is generally binary and string. The LayaAir engine has encapsulated the Socket and Byte classes for us, and sending and receiving data can be completed by combining the Byte class.



### 3.1 Laya.Soccet

In the LayaAir engine, `Socket` is the basic class we use WebSocket. `Socket` encapsulates HTML5 WebSocket, allowing full-duplex real-time communication between the server and the client, and allowing cross-domain communication. After establishing a connection, both the server and the Browser/Client Agent can actively send or receive text and binary data to each other. Let’s first understand the usage of `Socket`



#### 3.1.1 Connect server

There are three ways to connect to the server through Socket:

| way	| Description	|
| :--------------- | :----------------------------------------------------------- |
| Constructor parameter passing	| Connect immediately, such as new Socket("192.168.1.2",8899); note that the host parameter here does not have the ws prefix. |
| connect method	| Pass the url and port number to connect to the server; socket.connect("192.168.0.1.2", 8989); note that the host parameter here does not have the ws prefix. |
| connectByUrl method | Pass the entire url, such as socket.connectByUrl("ws://localhost:8989"); there is the ws prefix here. |



#### 3.1.2 Send data

Sending data is very simple. You only need to call the send function of Socket. The parameter can be string or ArrayBuffer.

- **Sending string format:**

```
this.socket.send("hello world");//This is the form of sending a string.
```

- **Send data in binary format:**

```typescript
//Write a byte
this.byte.writeByte(1);
//Write an int16 data
this.byte.writeInt16(20);
//Write a 32-bit floating point data
this.byte.writeFloat32(20.5);
//Write a string;
this.byte.writeUTFString("hello");
//Declare a temporary Byte type here
var by:Laya.Byte = new Laya.Byte();
//Set endian;
by.endian = Laya.Byte.LITTLE_ENDIAN;
//Write an int32 data
by.writeInt32(5000);
//Write a uint16 data
by.writeUint16(16);
//Write the temporary byte data into byte. Note here that by.buffer is written;
this.byte.writeArrayBuffer(by.buffer);
//Here, the byte array data is sent to the server through the socket.
this.socket.send(this.byte.buffer);
//Clear the data; make it easier to read and write next time;
this.byte.clear();
```

We saw above that the data we need is read into a Byte array through a byte array, and what is finally sent to the server is `byte.buffer`, which is an ArrayBuffer data type. It must be noted here that the parameter of send is ArrayBuffer. Many developers may not pay attention and directly pass it into Byte, resulting in incorrect data being sent. If it is written as `this.socket.send(this.byte)`; this is wrong, so you must pay attention to this.



#### 3.1.3 Receive data

The data received by the client from the server will be dispatched to the Event.MESSAGE listening function. The parameter of receiveHandler is the data sent back by the server. It may be a string or a binary ArrayBuffer. We don't need to read the received string, we can just use it directly. But if what is received is binary, we need to read it and convert it into the type we need.

```typescript
 private receiveHandler(msg: any = null): void {
   ///Receive data trigger function
   //.............Here we assume that what is received is a binary ArrayBuffer
   this.byte.clear();
   this.byte.writeArrayBuffer(msg);//Read the received binary data into a byte array for easy analysis.
   this.byte.pos = 0;//Set offset pointer;
   ////Start reading the data below. Read the data in order according to the data passed by the server.
   var a:number = this.byte.getByte();
   var b:number = this.byte.getInt16();
   var c:number = this.byte.getFloat32();
   var d:string = this.byte.getString();
   var e:string = this.byte.getUTFString();
 }
```



#### 3.1.4 Supported event types

What we commonly use is basically that the connection is successfully established, data is received, the connection is closed, scheduling after an exception occurs, etc.

```typescript
/**
 * Scheduled after the connection is successfully established.
 * @eventType Event.OPEN
 * */
/*[Event(name = "open", type = "laya.events.Event")]*/
/**
 * Scheduled after receiving data.
 * @eventType Event.MESSAGE
 * */
/*[Event(name = "message", type = "laya.events.Event")]*/
/**
 * Dispatched after the connection is closed.
 * @eventType Event.CLOSE
 * */
/*[Event(name = "close", type = "laya.events.Event")]*/
/**
 * Scheduling after an exception occurs.
 * @eventType Event.ERROR
 * */
/*[Event(name = "error", type = "laya.events.Event")]*/
```



#### 3.1.5 How to use it in code

Let’s give a simple WebSocket code example for sending and receiving data:

```typescript
    private connect(): void {
    
   	 //Create Socket object
   	 this.socket = new Socket();
   	 
   	 //Establish a connection to the server
   	 this.socket.connectByUrl("ws://echo.websocket.org:80");
   	 
   	 //Indicates the data in the buffer that needs to be sent to the server
   	 this.output = this.socket.output;
   	 
   	 //Add listening event
   	 this.socket.on(Event.OPEN, this, this.onSocketOpen);
   	 this.socket.on(Event.CLOSE, this, this.onSocketClose);
   	 this.socket.on(Event.MESSAGE, this, this.onMessageReveived);
   	 this.socket.on(Event.ERROR, this, this.onConnectError);
    }

    //Connection established successfully callback
    private onSocketOpen(e: any = null): void {
   	 console.log("Connected");

   	 // send string
   	 this.socket.send("demonstrate <sendString>");

   	 //Send using output.writeByte
   	 var message: string = "demonstrate <output.writeByte>";
   	 for (var i: number = 0; i < message.length; ++i) {
   		 // Directly write data in the buffer
   		 this.output.writeByte(message.charCodeAt(i));
   	 }
   	 
   	 //Send the data in the buffer to the server
   	 this.socket.flush();
    }

    //Event callback after the connection is disconnected
    private onSocketClose(e: any = null): void {
   	 console.log("Socket closed");
    }

    //Event callback when data is received
    private onMessageReveived(message: any = null): void {
   	 console.log("Message from server:");
   	 if (typeof (message) == 'string') {
   		 console.log(message);
   	 }
   	 else if (message instanceof ArrayBuffer) {
   		 console.log(new Byte(message).readUTFBytes());
   	 }
   	 // Clear cached data sent from the server
   	 this.socket.input.clear();
    }

    //Event callback after exception occurs
    private onConnectError(e: Event = null): void {
   	 console.log("error");
    }
```



### 3.2 Laya.Byte binary reading and writing

In development projects, binary operations are indispensable. In the HTML5 era, binary support has made great breakthroughs. However, the API is cumbersome and inconvenient for developers to develop projects. In the era of web games, the binary array ByteArray of ActionScript 3.0 has complete functions and the API operation is simple and easy to understand. Therefore, LayaAir's Byte takes over the characteristics of HTML5's TypedArray while referring to ByteArray. Let’s take a look at the main uses

#### 3.2.1 Common methods

- **Construction method**

  parameter:

  `length`: length

  When the length parameter is passed in, an internal array buffer is created, the size of the buffer is the length size passed in.

  `typedArray`: typed array

  When an arbitrary typed array object (typedArray) containing elements of any type (such as **Int32Array)** is passed as a parameter, typeArray is copied to a new typed array. Each value in typeArray is converted according to the constructor before being copied to the new array. The newly generated typed array object will have the same length as the passed array (for example, if the original typeArray.length==2, then the length of the newly generated array is also 2, but each item in the array has been converted. ).

  `ArrayBuffer`: Binary data buffer.

  The above three methods can instantiate a Byte and create binary data according to different parameters.

  ```typescript
  //Instantiate a binary array Byte
  var byte:Laya.Byte = new Laya.Byte();
  //Or pass in a typed array
  var uint8Byte:Uint8Array = new Uint8Array(10);
  var byte:Laya.Byte = new Laya.Byte(uint8Byte);
  //Or pass in an ArrayBuffer type
  var buffer:ArrayBuffer = new ArrayBuffer(20);
  var byte:Laya.Byte = new Laya.Byte(buffer);
  ```

- **writeArrayBuffer**(arraybuffer:*, offset:number = 0, length:number = 0):void

  Writes the specified binary buffer data. Specify the offset and length of the data, as follows:

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  var byte1:Laya.Byte = new Laya.Byte();
  byte1.writeFloat32(20.0);//Write a four-byte floating point number
  byte1.writeInt16(16);//Write a two-byte integer
  byte1.writeUTFString("hell world");//Write a string;
  byte.writeArrayBuffer(byte1.buffer,6);//Read the data of byte1 into byte starting from the sixth byte. Omit the floating point number 20.0 and the integer 16
  byte.pos = 0;//
  console.log(byte.readUTFString())//Read the string from byte.
  ```

- **Read data**

  `getByte():number`

  Read signed bytes from a byte stream.

  `getInt16():number`

  Reads an Int16 value from the byte stream at the current byte offset.

  `getInt32():number`

  Reads an Int32 value from the byte stream at the current byte offset.

  `getFloat32():number`

  Reads an IEEE 754 single-precision (32-bit) floating-point number from the byte stream at the current byte offset.

  `getFloat32Array(start:number, len:number)any`

  Reads the specified length of data from the specified location to create a Float32Array object and returns this object.

  `getFloat64():number`

  Reads an IEEE 754 double-precision (64-bit) floating-point number from the byte stream at the current byte offset.

  `getInt16():number `

  Reads an Int16 value from the byte stream at the current byte offset.

  `getInt32():number`

  Reads an Int32 value from the byte stream at the current byte offset.

  `getUint8():number`

  Reads a Uint8 value from the byte stream at the current byte offset.

  `getUint16():number`

  Reads a Uint16 value from the byte stream at the current byte offset.

  `getUint32():number`

  Reads a Uint32 value from the byte stream at the current byte offset.

  `getInt16Array(start:number, len:number):any`

  Reads data of the specified length from the specified location to create an Int16Array object and returns this object.

  `getString():string`

  Read character value.

  `getUTFBytes(len:number = -1):string `

  The read string must be the string written by the writeUTFBytes method.

  `getUTFString():string `

  Read UTF-8 string.

 

- **data input**

  `writeByte(value:number):void` writes a byte in the byte stream.

```typescript
 var byte:Laya.Byte = new Laya.Byte();
 byte.writeByte(10);//between 0-255
```

 `writeFloat32(value:number):void` writes a Float32 value at the current byte offset. The range is $\left[-2^{128}, 2^{127}\right]$, which is approximately -3.4E38—3.4E+38.

```typescript
var byte:Laya.Byte = new Laya.Byte();
byte.writeFloat32(10.021);
```

 `writeFloat64(value:number):void` writes a float64-bit value. The value range is -1.7E308~1.7E+308.

 `writeInt16(value:number):void` writes an Int16 value at the current byte offset. The range is -32768 to +32767.

```typescript
var byte:Laya.Byte = new Laya.Byte();
byte.writeInt16(120);
```

 `writeInt32(value:number):void` writes an Int32 value at the current byte offset. A signed integer between -2,147,483,648 and +2,147,483,647.

```typescript
 **writeUint16**(value:number):void writes a Uint16 value at the current byte offset.
```

 `writeUint32(value:number):void` writes a Uint32 value at the current byte offset.

 `writeUint8(value:number):void` writes a Uint8 value at the current byte offset.

 `writeUTFBytes(value:string):void` writes a string. The string written by this method must be read using the readUTFBytes method.

 `writeUTFString(value:string):void` writes a UTF-8 string to the byte stream.



- `clear():void` clears data.

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.writeInt16(120);
  byte.pos =0;//Return the reading position to zero.
  ```

 

- `getSystemEndian():string[static]` Gets the byte storage order of the system.

  ```typescript
  console.log(Laya.Byte.getSystemEndian());//Print the byte order of the system
  ```



#### 3.2.2 Properties

- `BIG_ENDIAN : string= bigEndian[static]` means that the most significant byte of a multibyte number is at the beginning of the byte sequence.

- `LITTLE_ENDIAN : string= littleEndian[static]` means that the least significant byte of a multi-byte number is at the beginning of the byte sequence.

- `pos `number currently reads the position.

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.writeInt16(120);
  byte.pos =0;//Return the reading position to zero.
  ```

- `length: number` Length in bytes.

- `endian : string` byte order.

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.endian = Laya.Byte.BIG_ENDIAN; //Set to big endian;
  ```

- `bytesAvailable : number[read-only]`The number of bytes of data that can be read from the current position of the byte stream to the end.

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.writeFloat32(20.0);
  byte.writeInt16(16);
  byte.writeUTFString("hell world");
  byte.pos = 6;
  console.log(byte.bytesAvailable)
  ```



#### 3.2.3 Code Demonstration

Below we use a complete code to demonstrate the application of this class. For example, in network connection, we receive and send network messages.

```typescript
var msg:any ={name:"xxx",age:18,weight:65.5,height:175};
var byte:Laya.Byte = new Laya.Byte();
//instantiate byte array
byte.endian = Laya.Byte.LITTLE_ENDIAN;
//Set the big and small ends
byte.writeUTFString(msg.name);
//data input
byte.writeByte(msg.age);
byte.writeFloat32(msg.weight);
byte.writeInt16(msg.height);
```

Take a look at the output:

```typescript
//Set pos to 0 and start reading from the beginning in the order of writing.
byte.pos = 0;
console.log(byte.getUTFString());
console.log(byte.getByte());
console.log(byte.getFloat32());
console.log(byte.getInt16());
```



#### 3.2.4 Typed array

Laya's byte package is a typed array. Developers can refer to mdn's official API description. to expand the application of your own project.

- [DataView](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView) view provides a view that matches the platform's order of bytes in memory (endian ) Independent low-level interface for reading and writing multi-numeric types from [`ArrayBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer).
- [Uint8Array](https://developer.mozilla.org/zh_CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) The array type represents an 8-bit unsigned integer array, and the content is initialized to 0 when created. After creation, the elements in the array can be referenced as objects or using array subscript indexing.
- **Int8Array**: Type array represents an array of two's complement 8-bit signed integers. Contents are initialized to 0. Once created, you can use the object's methods to reference elements in the array, or use standard array indexing syntax.
- **Int16Array()**; type array represents a two's complement 16-bit signed array.
- **Uint16Array()**; type array represents a two's complement 16-bit unsigned array
- **Int32Array()**; type array represents a two's complement 32-bit signed array
- **Uint32Array()**; type array represents a two's complement 32-bit unsigned array
- **Float32Array()**; type array represents a 32-bit floating point array.
- **Float64Array()**; type array represents a 64-bit floating point array.



## 4. Use of ProtocolBuffer

**protocolbuffer** (hereinafter referred to as PB) is a data exchange format of Google. It is independent of language and platform. Similar to data representation languages ​​such as XML and JSON, ProtocolBuffer is a flexible, efficient, and automatic method for serializing structured data. The format is somewhat similar to XML. You can define the data format yourself. It is a binary format that allows you to use A canonical language defines a pattern.

As a protocol format for network communication, ProtocolBuffer is a very popular method now. Let's take a look at it.



### 4.1 Message Definition

Here is a simple example, which is a very simple request message format.

```typescript
// awesome.proto
package awesomepackage;

//Specify proto version
syntax = "proto3";

//message contains multiple types of fields
message AwesomeMessage {
	string awesome_field = 1; // becomes awesomeField
}
```

In the above example, the type of fields is string. Of course, you can also specify more complex fields, such as enumeration type enum, or nested message type.

After the above Message is defined, we save the protocol file to the project directory

"assets/res/protobuf/awesome.proto";



### 4.2 Add protobuf class library to the project

We can download the latest `protobuf` class library from https://github.com/protobufjs/protobuf.js and put it in the bin directory of the project, and reference protobuf.js in index.html

```typescript
    <script type="text/javascript" src="protobuf.js"></script>
```



### 4.3 Load protocol file

`protobuf` class library, loads protocol files through the load method

```typescript
/**
 * Loads one or multiple .proto or preprocessed .json files into a common root namespace and calls the callback.
 * @param {string|string[]} filename One or multiple files to load
 * @param {Root} root Root namespace, defaults to create a new one if omitted.
 * @param {LoadCallback} callback Callback function
 * @returns {undefined}
 * @see {@link Root#load}
 */
function load(filename, root, callback) {
	if (typeof root === "function") {
    	callback = root;
    	root = new protobuf.Root();
	} else if (!root)
    	root = new protobuf.Root();
	return root.load(filename, callback);
}
```



### 4.4 Message method

- Message.verify(message: Object): null|string

Verify whether a Message object meets the requirements for a valid message.

- Message.create(properties: Object): Message

Creates a new message instance from a set of Javascript data that meets the requirements for a valid message.

- Message.encode(message: Message|Object , writer: Writer): Writer

Encode the Message object for network communication transmission.

- Message.decode(reader: Reader|Uint8Array): Message

In network communication transmission data, the Mesaage object is obtained by decoding.

- Message.toObject(message: Message , options: ConversionOptions): Object

Convert Message object data to a set of Javascript data.



### 4.5 Code Example

```typescript
	onAwake(): void {
    
   	 var resPath: string = "assets/res/protobuf/awesome.proto";
   	 //Load protobuf file
   	 this.ProtoBuf.load(resPath, this.onAssetsLoaded);
    }

    private onAssetsLoaded(err: any, root: any): void {
   	 if (err)
   		 throw err;

   	 // Get a Message message type
   	 var AwesomeMessage: any = root.lookupType("awesomepackage.AwesomeMessage");

   	 console.log(AwesomeMessage);

   	 // Initialization data
   	 var payload: any = { awesomeField: "AwesomeString" };
   	 console.log(payload);

   	 //Verify whether the data is valid
   	 var errMsg: any = AwesomeMessage.verify(payload);

   	 // If there is an exception, throw an exception and terminate
   	 if (errMsg)
   		 throw Error(errMsg);

   	 //Create the entity of Message
   	 var message: any = AwesomeMessage.create(payload);
   	 console.log(message);

   	 //Compile Message entity into Buffer data format and wait for sending data
   	 var buffer: any = AwesomeMessage.encode(message).finish();
   	 console.log(buffer);

    }
```






