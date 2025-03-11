# LayaNativeReview



LayaNative is a complete development solution for the development, testing, and release of mobile native apps for the LayaAir engine, but it is not limited to the LayaAir engine. Based on LayaPlayer as the core runtime, LayaNative uses the reflection mechanism and channel docking solution to provide developers with secondary opening and channel docking on the native App, and provides testers and construction tools to package and publish HTML5 projects for developers. Conveniently turned into a native App.




## **LayaNative contains the following content:**


### 1. Tester:
After downloading and installing the tester, scanning the URL QR code helps developers quickly see the running effect on the mobile terminal, saving a lot of time in repeated packaging and testing;

 

### 2. Build tools:
The build tool can help developers quickly build mobile APP projects, and then use Android Studio, Eclipce, XCode and other development tools to open -> build -> run;



### 3. Reflection mechanism:
Through the reflection mechanism, developers can realize mutual calls between JavaScript and native languages ​​​​(Android/Java or iOS/Objective-C). Through the reflection mechanism, developers can easily expand the application twice;



### 4. In the channel docking tool (conchMarket):
The channel docking tool embeds common channel docking APIs, such as: login, sharing, recharge, friend relationship chain, etc.;



### 5. LayaPlayer：
LayaPlayer is the core part of LayaNative. It is a cross-platform engine based on JavaScript script engine + openGLES hardware-accelerated rendering. It accelerates multimedia applications, games and other products based on HTML5 and WEBGL by extremely optimizing the memory and rendering process. Its performance is comparable to native Native-APP. LayaPlayer is written in C++ language and can be embedded in a browser or operating system to run, or it can run independently.



### 6. The principle and development process of LayaNative
(1) Projects developed using LayaAir are ready to be released into app versions (ios or android).
(2) LayaNative will use the core engine LayaPlayer for acceleration.
(3) Developers can use the tester to quickly install it on mobile devices for simple testing.
(4) Finally, build the ios or android project through the command line or IDE, compile and execute it.
(5) If you need to publish to major channels (for example: Baidu, 360, AppStroe, Google, etc.), you need to carry out secondary development through the reflection mechanism (ie: docking the channel's SDK, logging in, recharging, sharing, etc.).
(6), finally build it into an app for installation, testing, and release.

The process is shown in Figure 1:

​    ![blob.png](img/1.png)
Figure (1)





