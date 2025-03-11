# Use reflection mechanism to implement secondary development

	LayaNative helps developers facilitate secondary development by providing a reflection mechanism. Let’s use an example to learn how to carry out secondary development.

## 1. Call static function

	Using LayaNative, you can call static functions written in the mobile terminal's native development language (Java under Android, Objective-C under iOS) in the JavaScript layer.

####	1.1 JavaScript layer:

	How to call the JavaScript layer:

```javascript

  var os = conchConfig.getOS();
  was bridge;
  var obj = {};
  if (os == "Conch-ios") {
  	bridge = PlatformClass.createClass("JSBridge");//Create a script proxy
  }
  else if (os == "Conch-android") {
	//Requires complete class path, note the difference from iOS
	bridge = PlatformClass.createClass("demo.JSBridge");//Create a script proxy
  }

  if (os == "Conch-ios") {
	//For iOS, pay attention to the function signature and the difference from Android.
	alert(bridge.call("testString:","hello"));
	alert(bridge.call("testNumber:",256.0));
	alert(bridge.call("testBool:",false));
	obj.value = "Hello OC!";
	bridge.callWithBack(function(value) {
  	var obj = JSON.parse(value)
  	alert(obj.value);
  	},"testAsyncCallback:", JSON.stringify(obj));
  }
  else if (os == "Conch-android") {
	alert(bridge.call("testString","hello"));
	alert(bridge.call("testNumber",256.0));
	alert(bridge.call("testBool",false));
	obj.value = "Hello Java!";
	bridge.callWithBack(function(value) {
  	var obj = JSON.parse(value)
  	alert(obj.value);
	},"testAsyncCallback",JSON.stringify(obj));
  }

```


#### 	1.2 Android/Java layer

	Add the following functions to class JSBridge:

```javascript
	public static String testString(String value) {
    	Log.d("JSBridge", "java: " + value);
    	return "LayaBox";
	}
	public static double testNumber(double value) {
    	Log.d("JSBridge", "java: " + value);
    	return 512;
	}
	public static boolean testBool(boolean value) {
    	Log.d("JSBridge", "java: " + value);
    	return value ? false : true;
	}
	public static void testAsyncCallback(String json) {
    	//js thread
    	try {
        	JSONObject root = new JSONObject(json);
        	Log.d("JSBridge", "java: " + root.getString( "value" ));
    	} catch (JSONException e) {
        	e.printStackTrace();
    	}
    	m_Handler.post(
            	new Runnable() {
                	public void run() {
                    	//ui thread update ui
                    	JSONObject obj = new JSONObject();
                    	try {
                        	obj.put("value", "Hello JS!");
                    	} catch (JSONException e) {
                        	e.printStackTrace();
                    	}
                    	ExportJavaFunction.CallBackToJS(JSBridge.class,"testAsyncCallback", obj.toString());
                	}
            	});
	}
```

#### 	2.1.4 iOS/OC layer

	Add the following functions to class JSBridge:

```javascript
	+(NSString*)testString:(NSString*)value
	{
  	NSLog(@"OC: %@",value);
  	return @"LayaBox";
	}
	+(NSNumber*)testNumber:(NSNumber*)value
	{
  	NSLog(@"OC: %@",value);
  	return @512;
	}
	+(NSNumber*)testBool:(NSNumber*)value
	{
  	NSLog(@"OC: %d",value.boolValue);
  	return [NSNumber numberWithBool:value.boolValue ? NO : YES];
	}
	+(void)testAsyncCallback:(NSString*)json
	{
  	//js thread
  	NSError* error = nil;
  	NSData* jsonData = [json dataUsingEncoding:NSUTF8StringEncoding];
  	NSDictionary* dict = [NSJSONSerialization JSONObjectWithData:jsonData options:NSJSONReadingMutableContainers error:&error];
  	NSLog(@"OC: %@", [dict objectForKey:@"value"]);
  	dispatch_async(dispatch_get_main_queue(), ^{
      	//ui thread
      	NSError* error = nil;
      	NSDictionary* dic = [NSDictionary dictionaryWithObject:@"Hello JS!" forKey:@"value"];
      	NSData* jsonData = [NSJSONSerialization dataWithJSONObject:dic options:NSJSONWritingPrettyPrinted error:&error];
      	NSString* jsonStr = [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
      	[[conchRuntime GetIOSConchRuntime] callbackToJSWithClass:self.class methodName:@"testAsyncCallback:" ret:jsonStr];
  	});
  }

```
Notice:
	Function parameters only support basic types such as Boolean, floating point, and string, and return values ​​are supported. Native functions run in the script thread. To update the UI, you need to go to the UI thread and support asynchronous callback functions.
The suffix of the OC source file must be changed to .mm, and the OC method is a static class method and must use +.


  Through the above method, secondary development related to native code can be easily carried out.


## 3. Platform code (android/ios) actively executes js scripts

### 	3.1 IOS/OC executes JS script

```javascript
  [[conchRuntime GetIOSConchRuntime] runJS:@"alert('hello')"];
```

### 	3.2 Android/Java executes JS script

```javascript
  ConchJNI.RunJS("alert('hello world')");
```




