# Native Platform Communication

After supporting HarmonyOS, Linux, and Windows, a new interface for communication between and the native platform has been added.

## 1. Script Interface
```javascript
// Synchronous
postSyncMessage(eventName: string, data: string): string;
// Asynchronous
postAsyncMessage(eventName: string, data: string): Promise<string>;
```
A simple test case is as follows:  
```javascript
var ret = conch.postSyncMessage("syncMessage", "syncMessage from js");
alert(ret);
conch.postAsyncMessage("asyncMessage", "asyncMessage from js").then(function (data) {
alert(data);
})
```
## 2. Native Platform End Message Processing
### 1. HarmonyOS
Add message processing code in libSysCapabilities/src/main/ets/event/HandleMessageUtils.ts
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
    * @param cb Callback
    */
    static async handleAsyncMessage(eventName: string, data: string, cb: Function): Promise<void> {
        if (eventName == "asyncMessage") {
            cb("async message from platform");
        }
    }
```
### 2. Android
Add message processing code in app/src/main/java/demo/HandleMessageUtils.java
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
Add message processing code in HandleMessageUtils.mm  
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
### 4. Windows
The conchSetHandleMessageCallback function sets the callbacks for processing asynchronous and synchronous messages.  
conchSendHandleMessageResult passes the data back to the JS side based on the event name.  
See Runtime/x64/include/Exports.h for details.  
```c
CONCH_EXPORT void CONCH_CDECL conchSetHandleMessageCallback(handleSyncMessageCallback handleSyncMessageCb,
                                                            handleAsyncMessageCallback handleAsyncMessageCb);
CONCH_EXPORT void CONCH_CDECL conchSendHandleMessageResult(const char *eventName, const char *result);
```
Message processing  
```c
int WINAPI wWinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance, LPWSTR lpCmdLine, int nShowCmd)
{
    conchSetHandleMessageCallback(
        [](const char *eventName, const char *data) -> void {
            if (strcmp(eventName, "syncMessage") == 0)
            {
                conchSendHandleMessageResult(eventName, "sync message from platform");
            }
        },
        [](const char *eventName, const char *data) -> void {
            if (strcmp(eventName, "asyncMessage") == 0)
            {
                conchSendHandleMessageResult(eventName, "async message from platform");
            }
        });
    return conchMain(hInstance, hPrevInstance, lpCmdLine, nShowCmd);
}
```
### 5. Linux
Same as Windows
The conchSetHandleMessageCallback function sets the callbacks for processing asynchronous and synchronous messages.  
conchSendHandleMessageResult passes the data back to the JS side based on the event name.  
See Runtime/x86_64/include/Exports.h for details.  
```c
CONCH_EXPORT void CONCH_CDECL conchSetHandleMessageCallback(handleSyncMessageCallback handleSyncMessageCb,
                                                            handleAsyncMessageCallback handleAsyncMessageCb);
CONCH_EXPORT void CONCH_CDECL conchSendHandleMessageResult(const char *eventName, const char *result);
```
Message processing 
```c
int main(int argc, char *argv[])
{
    conchSetHandleMessageCallback(
        [](const char *eventName, const char *data) -> void {
            if (strcmp(eventName, "syncMessage") == 0)
            {
                conchSendHandleMessageResult(eventName, "sync message from platform");
            }
        },
        [](const char *eventName, const char *data) -> void {
            if (strcmp(eventName, "asyncMessage") == 0)
            {
                conchSendHandleMessageResult(eventName, "async message from platform");
            }
        });
    return conchMain(argc, argv);
}
```