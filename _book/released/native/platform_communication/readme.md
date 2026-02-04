# Native Language and JavaScript Communication

Sometimes, we need to extend some native language functions in Native. How can we communicate with the LayaAir engine's JS language project? This article will provide a comprehensive introduction.

# 1. Execution of JS-Side Scripts

### 1.1 Sending Messages from JS to Native

The script interface used in JS is as follows:

```javascript
//Synchronous
postSyncMessage(eventName: string, data: string): string;
//Asynchronous
postAsyncMessage(eventName: string, data: string): Promise<string>;
```

A simple test case in JS is as follows:

```javascript
var ret = conch.postSyncMessage("syncMessage", "syncMessage from js");
alert(ret);
conch.postAsyncMessage("asyncMessage", "asyncMessage from js").then(function (data) {
    alert(data);
})
```

### 1.2 Actively Executing JS-Side Scripts at the Native Side

iOS/OC executing JS scripts:

```javascript
  [[conchRuntime GetIOSConchRuntime] runJS:@"alert('hello')"];
```

Android/Java executing JS scripts:

```javascript
  ConchJNI.RunJS("alert('hello world')");
```

# 2. Native-Side Message Handling

Note: The native-side message handling functions in each code branch must return corresponding values or messages to avoid freezing.

### 1. HarmonyOS
Add message handling code in libSysCapabilities/src/main/ets/event/HandleMessageUtils.ts

```typescript
    /**
    * Synchronous event
    * @param eventName Event name
    * @param data Data
    */
    static handleSyncMessage(eventName: string, data: string): string {
        if (eventName == "syncMessage") {
            return "sync message from platform";
        }
        return "default sync result";
    }

    /**
    * Asynchronous event
    * @param eventName Event name
    * @param data Data
    * @param cb callback
    */
    static async handleAsyncMessage(eventName: string, data: string, cb: Function): Promise<void> {
        if (eventName == "asyncMessage") {
            cb("async message from platform");
        }
    }
```

### 2. Android
Add message handling code in app/src/main/java/demo/HandleMessageUtils.java

```java
    public static String handleSyncMessage(String eventName, String data) {
        Log.d(LOG_TAG, eventName +" " + data);
        if (eventName.equals("syncMessage")) {
            return "sync message from platform";
        }
        return "default sync result";
    }
    public static void handleAsyncMessage(String eventName, String data, HandleMessageCallback cb) {
        Log.d(LOG_TAG, eventName +" " + data);
        if (eventName.equals("asyncMessage")) {
            cb.callback("async message from platform");
        }
    }
```

### 3. iOS
Add message handling code in HandleMessageUtils.mm

```c
+(NSString*)handleSyncMessageWithEventName:(NSString*)eventName data:(NSString*)data {
    NSLog(@"%@ %@", eventName, data);
    if ([eventName isEqualToString:@"syncMessage"]) {
        return @"sync message from platform";
    }
    return @"default sync result";
}
+(void)handleAsyncMessageWithEventName:(NSString*)eventName data:(NSString*)data callback:(void (^)(NSString *))cb {
    NSLog(@"%@ %@", eventName, data);
    if ([eventName isEqualToString:@"asyncMessage"]) {
        cb(@"async message from platform");
    }
}
```
