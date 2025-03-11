
# About network status monitoring
Since the network environment of mobile devices is not stable, when the network changes, the project often needs to give users some prompts. There are two methods in LayaNative to obtain changes in the network environment.

## 1. Monitoring method

Developers can use registered listening functions to monitor network changes. The code is as follows:

The JS code is as follows
```javascript
if( conch )
{
	conch.setNetworkEvtFunction(function(type)
	{
    	alert(type)
	});
}
```
The AS code is as follows:
```javascript
if ( Render.isConchApp)
{
	Browser.window["conch"].setNetworkEvtFunction(function(type):void
	{
    	alert(type)
	});
}
```

**The return value class is int type**
```
NET_NO = 0;
NET_WIFI = 1;
NET_2G = 2;
NET_3G = 3;
NET_4G = 4;
NET_YES = 5;
```
**Tips**  
*1. Conch can only be called in the LayaNative environment. There is no definition of conch in the web version, so you need to check whether it exists.*
*2. Or you can use if(Render.isConchApp) to judge.*

## 2. Query method

Developers can also query the network status through active query. The code is as follows:

```javascript
if( conch )
{
	var nType = conch.config.getNetworkType();
}
```

**The return value class is int type**
```
NET_NO = 0;
NET_WIFI = 1;
NET_2G = 2;
NET_3G = 3;
NET_4G = 4;
NET_YES = 5;
```






