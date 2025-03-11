# About LocalStorage

LayaNative supports the use of LocalStorage, but there are format requirements. GetItem() and setItem() must be used to store and obtain values.

### Usage under AS

```java
//Storage the specified key name and key value, string type.
LocalStorage.setItem("LayaBox","H5 engine!");
//Get the value of the specified key name.
LocalStorage.getItem("LayaBox");
```



### Usage under JS and TS

```java
//Storage the specified key name and key value, string type.
Laya.LocalStorage.setItem("LayaBox","H5 engine!");
//Get the value of the specified key name.
Laya.LocalStorage.getItem("LayaBox");
```



### Incorrect usage:

The usage of the following js syntax is supported by PC browsers or mobile terminals (browser naked running), but is not supported by LayaNative.

```java
//Storage, not supported under LayaNative
localStorage.test = 100;
//Value, not supported under LayaNative
alert(localStorage.test);
```





