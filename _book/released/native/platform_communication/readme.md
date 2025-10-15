# Native and JavaScript Communication

Sometimes, we need to extend native functionalities in the platform layer. But how can these native modules communicate with the JavaScript code used in the LayaAir engine?
This document provides a complete overview of communication between **Native** and **JS**.

# 1. Executing Scripts on the JS Side

### 1.1 Sending Messages from JS to the Native Side

In JavaScript, you can use the following interfaces to send messages to the native environment:

```javascript
// Synchronous
postSyncMessage(eventName: string, data: string): string;

// Asynchronous
postAsyncMessage(eventName: string, data: string): Promise<string>;
```

Here is a simple JS test example:

```javascript
var ret = conch.postSyncMessage("syncMessage", "syncMessage from js");
alert(ret);

conch.postAsyncMessage("asyncMessage", "asyncMessage from js").then(function (data) {
    alert(data);
});
```

### 1.2 Executing JS Code from the Native Side

**iOS / Objective-C** executing JS code:

```objective-c
[[conchRuntime GetIOSConchRuntime] runJS:@"alert('hello')"];
```

**Android / Java** executing JS code:

```java
ConchJNI.RunJS("alert('hello world')");
```

# 2. Message Handling on the Native Side

### 1. HarmonyOS

Add message handling code in
`libSysCapabilities/src/main/ets/event/HandleMessageUtils.ts`:

```typescript
/**
 * Handle synchronous events
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
 * Handle asynchronous events
 * @param eventName Event name
 * @param data Data
 * @param cb Callback function
 */
static async handleAsyncMessage(eventName: string, data: string, cb: Function): Promise<void> {
    if (eventName == "asyncMessage") {
        cb("async message from platform");
    }
}
```

### 2. Android

Add message handling code in
`app/src/main/java/demo/HandleMessageUtils.java`:

```java
public static String handleSyncMessage(String eventName, String data) {
    Log.d(LOG_TAG, eventName + " " + data);
    if (eventName.equals("syncMessage")) {
        return "sync message from platform";
    }
    return "default sync result";
}

public static void handleAsyncMessage(String eventName, String data, HandleMessageCallback cb) {
    Log.d(LOG_TAG, eventName + " " + data);
    if (eventName.equals("asyncMessage")) {
        cb.callback("async message from platform");
    }
}
```

### 3. iOS

Add message handling code in `HandleMessageUtils.mm`:

```objective-c
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

Use the `conchSetHandleMessageCallback` function to register callback handlers for **synchronous** and **asynchronous** messages.
Use `conchSendHandleMessageResult` to send data back to the JS side based on the event name.
Refer to `Runtime/x64/include/Exports.h` for details.

```c
CONCH_EXPORT void CONCH_CDECL conchSetHandleMessageCallback(handleSyncMessageCallback handleSyncMessageCb,
                                                            handleAsyncMessageCallback handleAsyncMessageCb);
CONCH_EXPORT void CONCH_CDECL conchSendHandleMessageResult(const char *eventName, const char *result);
```

Message handling example:

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

The implementation is the same as Windows.
Use `conchSetHandleMessageCallback` to set up the callbacks for handling messages,
and use `conchSendHandleMessageResult` to pass results back to the JS side.
Refer to `Runtime/x86_64/include/Exports.h` for details.

```c
CONCH_EXPORT void CONCH_CDECL conchSetHandleMessageCallback(handleSyncMessageCallback handleSyncMessageCb,
                                                            handleAsyncMessageCallback handleAsyncMessageCb);
CONCH_EXPORT void CONCH_CDECL conchSendHandleMessageResult(const char *eventName, const char *result);
```

Message handling example:

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
