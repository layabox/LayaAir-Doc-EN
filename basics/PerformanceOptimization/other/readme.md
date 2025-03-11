# Other optimization strategies

### 1. Reduce the number of particles used

Since particles are vector drawing, using a large number of particles puts a lot of pressure on the CPU. In the Canvas mode of the mobile platform, try not to use particles;

GPU computing can be used in WebGL mode, which can reduce CPU pressure, but it should be controlled as much as possible to reduce usage.



### 2. Canvas mode minimizes the use of attributes such as rotation, scaling, and alpha.

In Canvas mode, try to reduce the use of rotation, scaling, alpha and other attributes, which will consume performance.

If you want to use it, it is recommended to use it in WebGL mode;



### 3. Do not create objects and complex calculations in the Timer loop

Since the `loop()` and `frameLoop()` methods of Timer will be continuously executed in a loop, when creating objects and complex calculations, it will cause a lot of performance consumption in the loop. Therefore, try not to create objects in the loop as much as possible. Objects and complex calculations.



### 4. Use autoSize and getBounds as little as possible

`autoSize()` and `getBounds()` require a lot of calculations and have a great impact on performance, so use them as little as possible.



### 5. The execution of functions caught by try catch will become very slow.

Try to reduce the use of `try catch` in the project. The execution of functions caught by `try catch` will become very slow.

 

