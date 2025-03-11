

# Take over android back button
These two functions can be used in LayaNative, conch.setOnBackPressedFunction(onBack) and conch.exit(), to take over the processing of pressing the "back key". After taking over setOnBackPressedFunction, this function will be executed when the user presses the back key.

Once this function is called, the default function of pressing twice to exit in the engine is blocked. At this time, if you want to exit the application, you can do so by calling the exit() function.

**Tips**  
*1. Conch can only be called in the LayaNative environment. There is no definition of conch in the web version, so you need to check whether it exists.*
*2. Only the Android version of LayaNative has these two functions.*


js example is as follows:
```javascript
var n=3;
if(window.conch){
	window.conch.setOnBackPressedFunction(()=>{
    	console.log('press back '+n);
    	if(n-- <=0){
        	window.conch.exit();
    	}
    	else{
        	//User's own code, such as returning to the upper page
    	}
	});
}
```

