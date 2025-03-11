## LayaNative’s resource update method
After the game is released, you will inevitably encounter update problems. The update here refers to the fact that after the game packaged with LayaNative is released, you want to modify some of the client's code, pictures and other data because of bug fixes or added functions.
Currently LayaNative supports two resource update methods:

### 1. User-invisible updates (recommended).

This is an ongoing, ongoing update. This method is in line with the idea of ​​updating web pages: only when a certain resource needs to be used, the resource update process will be triggered. This broken update mechanism allows users to enter the game immediately and complete the update without even realizing it.
This update is based on LayaNative's DCC mechanism. For an introduction and usage of LayaDCC, see [here](../LayaDcc_Tool/readme.md).

### 2. Visible to users, centralized updates before entering the game.

Most traditional app update methods are to check whether an update is needed as soon as it comes up. If an update is required, download a large zip file and update it as a whole. The maintenance cost of this kind of update is high, users need to wait for a long time to enter the game, and it also clearly violates Apple's policy prohibiting hot updates. Its advantage is that users can update where there is wifi and play in places where there is no wifi, avoiding wasting data traffic when there is no wifi.
Although LayaNative does not directly support this kind of update, it can also achieve this function through the following interfaces (note that these interfaces are internal interfaces and may change in the future):
* Supports downloadBigFile, a large file download function that can be resumed at breakpoints. (Be careful not to use XMLHttpRequest to download large files, because in this way LayaNative will save the results in memory first, so large files may cause the memory to explode, and this function can be saved at any time.)

```javascript
	/**
	* @param url remote address
	* @param local save to local file
	* @param onprog Download progress callback
	* @param oncomp download completion callback
	* @param trynum number of retries (0 unlimited retries)
	* @param opttimeout timeout (recommended to be large enough)
	*/
	declare var downloadBigFile:(url:string,
    	local:string,
    	onprog:(total:number,now:number,speed:number)=>boolean,oncomp:(curlret:number, httpret:number)=>void,
    	trynum:number,
    	opttimeout:number)=>void;
    
```
* ZipFile class for processing zip files

```javascript
	interface ZipFile{
    	setSrc(src:string):boolean;
    	/**
    	* Traverse the files in the zip.
    	* id:
    	* name: file name, including path
    	* dir: whether it is a directory
    	* sz: file size
    	*/
    	forEach(func:(id:number,name:string,dir:boolean,sz:number)=>void):void;
    	/**
    	* Read the contents of the file in the zip and return an ArrayBuffer
    	*/
    	readFile(id:number):ArrayBuffer;
    	close():void;
   	 
    	new ():ZipFile;
	}
    
	declare var ZipFile:ZipFile;    
```
* The function of manually updating dcc cache.

```javascript
	interface AppCache{
    	...
    	/**
    	* Update a file in the dcc cache
    	* @param nameid The id of the updated file.
    	* Path rule: /, indicating the app root directory. For example: hashstr('/index.html'), do not take parameters. If you take parameters - hashstr('/aa/bb.html?ff=2'), no one will be able to find this file.
    	* @param chksum Check code, if 0, this function calculates it by itself. If it is external version control, this is the version number after hashstr.
    	* @param buf ArrayBuffer file content.
    	* @param extversion whether to use external version number
    	* @return boolean If true is returned, it means the update is successful, otherwise, it means the check code is inconsistent, that is
    	*  	You need to update dcc before it can work.
    	*/
    	updateFile(nameid:number,chksum:number,buf:ArrayBuffer,extversion:boolean):boolean;   	 
    	...
	}
```

Through these functions, a centralized update function can be implemented on layaDCC. For example, LayaNative provides an encapsulated update function updateByZip:
​    
```javascript
	/**
	* Update with zip.
	* @param {string} url zip download address
	* @param {(event:string,downloadPercent:number,curfile:string)=>void} onEvent event callback.
	*  	The event is the event name. Possible events are as follows:
	*      	'downloading' is downloading, and downloadPercent has a value at this time.
	*      	'downloadError' download error
	*      	'downloadOK' Download successful.
	*      	'updating' is being updated. At this time, curfile has a value, indicating the file being updated.
	*      	'updateError' curfile update error. Because curfile is not in the dcc list, or the file content is inconsistent with the dcc content. A small number of update errors can be ignored because they will still be downloaded during actual use.
	*      	'unknownError'
	* @param {function(localfile:string):void} onEnd update completion callback
	*/
	function  updateByZip(url, onEvent, onEnd)
```
The implementation code of this function is in the engine’s index.js. So if you have special needs, you can also refer to this function to implement your own update function.
​    
It should be noted that this function actually only downloads the zip and then updates each file in the zip to the cache. When actually using it, you have to implement version management, interface, download progress prompts and other functions yourself. In order to implement these functions, you may need a local interface for reading and writing files. You can use the following global function (also an internal interface, which may change):
​    
```javascript
	declare var fs_readFileSync:(file:string)=>ArrayBuffer;    
	declare var fs_writeFileSync:(file:string,data:string|ArrayBuffer)=>boolean;
	declare var readFileSync:(file:string,encode:string)=>string;//This directly returns a string.
```
If you need to obtain the cache path, you need the appcache object inside LayaNative:
​    
```javascript
 	var cachepath = window.appcache.getCachePath()
```



