# Display the Current Location Using Baidu Map

> This section demonstrates step by step how to mark the current location on Baidu Map using watchPosition(). The watchPosition method comes from the Geolocation API. **Before learning this section, please read the Geolocation [basic documentation](../geolocation/readme.md) or the Geolocation [API documentation](https://layaair.com/3.x/api/Chinese/index.html?version=3.0.0&type=Core&category=device&class=laya.device.geolocation.Geolocation).**
>

Before starting, it is necessary to introduce the script file of Baidu Map in index.html. This URL can be obtained for free on the official website of Baidu Map. The URL used in the demonstration is [http://api.map.baidu.com/api?v=2.0&ak=LIhOlvWfdiPYMCsK5wsqlFQD8wW4Bfy6](http://api.map.baidu.com/api?v=2.0&ak=LIhOlvWfdiPYMCsK5wsqlFQD8wW4Bfy6). The code added in the `<body> </body>` tag is as follows:

```html
<script src="https://api.map.baidu.com/api?v=2.0&ak=LIhOlvWfdiPYMCsK5wsqlFQD8wW4Bfy6&s=1&callback=appendCode"></script>
```

### **1. introduce the member variables:**

```typescript
// Baidu Map
private map: any;// Map reference
private marker: any;// Map marker
private BMap: any = Laya.Browser.window.BMap;// Baidu Map namespace
private convertor: any;// Coordinate conversion interface
private mapDiv: any;// Div container containing Baidu Map
```

### 2. then the constructor:

```typescript
    constructor() {
        Laya.init(1, 1);

        this.init();
    
        // Use high-precision location
        Laya.Geolocation.enableHighAccuracy = true;
        Laya.Geolocation.watchPosition(Laya.Handler.create(this, this.updatePosition), Laya.Handler.create(this, this.onError));
    
        // Bind the scope of convertToBaiduCoord
        this.convertToBaiduCoord = this.convertToBaiduCoord.bind(this);
    }
}
```

Since this example does not need to use the display elements of LayaAir, the stage size is set to 1. The initialization of the Baidu Map interface is placed in init(). Then, it listens for changes in the device position. Finally, it should be noted that the function convertToBaiduCoord() is to convert the obtained coordinates to Baidu Map coordinates. Since it is used as a parameter of convertor.translate(), the scope will be changed when triggered. Therefore, the scope of this function is bound here.

##### 2.1 init function:

```typescript
private  init(): void {
  this.mapDiv = Laya.Browser.createElement("div");
  Laya.Browser.document.body.appendChild(this.mapDiv);

  // Adapt to the window size
  this.refit();
  Laya.stage.on(Laya.Event.RESIZE, this, this.refit);

  // Initialize the map
  this.map = new this.BMap.Map(this.mapDiv);

  // Disable some interactions
  //this.map.disableDragging();
  this.map.disableKeyboard();
  this.map.disableScrollWheelZoom();
  this.map.disableDoubleClickZoom();
  this.map.disablePinchToZoom();
  // Initial location Beijing, zoom factor 15
  this.map.centerAndZoom(new this.BMap.Point(116.32715863448607, 39.990912172420714), 15);

  // Create a marker
  this.marker = new this.BMap.Marker(new this.BMap.Point(0, 0));
  this.map.addOverlay(this.marker);
}
```

The init() function initializes Baidu Map. Most interaction functions are closed, leaving only the dragging of the map. The initial location of the map is in Beijing, and the zoom factor is 15. And a map marker is added.

##### 2.2 refit function:

```typescript
private  refit(): void {
  this.mapDiv.style.width  =  Laya.Browser.width  / Laya.Browser.pixelRatio  +  "px";
  this.mapDiv.style.height  = Laya.Browser.height  / Laya.Browser.pixelRatio  +  "px";
}
```

The refit() function makes the Baidu Map fill the entire window. Since the resize event is listened to, the window will be refilled when the window is resized.

##### 2.3 updatePosition function:

```typescript
// Update the device position
private  updatePosition(p: Laya.GeolocationInfo): void {
  // Convert to Baidu Map coordinates
  var  point:any = new this.BMap.Point(p.longitude,  p.latitude);
  // Convert the original coordinates to Baidu coordinates. For some device browsers, the obtained coordinates may be Google coordinates. At this time, the third parameter should be changed to 3 for it to be correct.
  this.convertor.translate([point],  1,  5,  this.convertToBaiduCoord);
}
```

The updatePosition() is the trigger function of Geolocation.watchPosition(). After each position change is detected, the obtained original coordinates need to be converted to Baidu coordinates to display the correct position on Baidu Map.

Note that for some device browsers, the obtained coordinates may be Google coordinates. At this time, the third parameter of convertor.translate is not 5 but 3.

##### 2.4 convertToBaiduCoord function:

```typescript
// Convert the original coordinates to Baidu coordinates
private  convertToBaiduCoord(data: any): void {
  if  (data.status  ==  0) {
    var  position: any  =  data.points[0];
    // Set the position of the marker
    this.marker.setPosition(position);

    this.map.panTo(position);
  }
}
```

After the conversion is completed, set the position of the marker and pan the viewport to the center of the marker.

##### 2.5 onError function:

```java
private onError(e: any): void {
        var errType: string;
        if (e.code = Laya.Geolocation.PERMISSION_DENIED)
            errType = "Permission Denied";
        else if (e.code == Laya.Geolocation.POSITION_UNAVAILABLE)
            errType = "Position Unavailable";
        else if (e.code == Laya.Geolocation.TIMEOUT)
            errType = "Time Out";
        alert('ERROR(' + errType + '): ' + e.message);
    }
```

After completing the above steps, you can view the effect in the browser on the device. If the position is incorrect, try treating the obtained coordinates as Google coordinates. Note that the security restrictions of the browser itself may require the user to manually allow the web page to use the geographical location, or Chrome requires an https protocol address to be able to use the geographical location.