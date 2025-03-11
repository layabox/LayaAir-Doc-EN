## Component development of LayaAir

> Author: charley        	updated date: 2020-11-17

In fact, LayaAir 2.0 mentioned when it was released that compared to the 1.0 engine, support for component development is one of the very important new features of the 2.0 engine and IDE. However, until today, there are still some developers who do not fully understand the component development of the LayaAir engine. This article briefly introduces the component development of LayaAir.

### 1. What is component development?

There are actually many different understandings of component development. Here I will explain my understanding, hoping it will be useful to everyone.

As long as programmers have some work experience, they will basically encounter refactoring or take over the maintenance of other people's project code. It will be very painful to encounter code with a high degree of coupling. Once the need for refactoring arises, it will be difficult to refactor. And during maintenance, in order to correct a BUG somewhere, another new BUG may have just been repaired, causing various maintenance difficulties.

Component development can alleviate all the above pains.

Because the core idea of ​​component-based development is decoupling and **single responsibility**.

Let's first understand componentization literally. Groups are combinable, and pieces are single, that is, they can be independent. Therefore, the design idea of ​​componentization is that it must not be coupled with other modules, because if coupled, it will be difficult to become independent.

If decoupling is the core design idea of ​​component development, then single responsibility is the guiding method to achieve this core design idea.

When a class has too many responsibilities, there will inevitably be the possibility of coupling. Therefore, making the responsibilities of each class as clear and single as possible will avoid coupling.

2D sample project (block game) created through LayaAirIDE. It is also an example of component-based development design thinking, taking into account understanding and single responsibility. For example, the bullet script is only responsible for its own removal and recycling, and there is no need to consider what to do with the box if it touches the box. The box will have its own script to handle the box's collision and rebound, degradation, death explosion, etc. after being hit by a bullet. Therefore, no matter how many bullets or boxes there are in the scene, the script of each bullet or box is only responsible for the node to which it belongs.

After understanding the basic concept of component development, the next step is to use the LayaAir engine to develop based on component design ideas.

### 2. Script components and life cycle

The LayaAir engine itself provides a large number of components, such as UI components and physical components. We also allow developers to add script components to nodes to achieve individual control of nodes.

In order to better allow script classes to manage their own nodes, the engine also provides a large number of life cycle methods.

> Life cycle is a series of important nodes from birth to death, such as activation of nodes, adding to the stage, refreshing, collision, destruction of each frame, etc. The logic within the life cycle method will be automatically executed when the corresponding conditions are encountered.

#### Mainline life cycle method:

| Lifecycle methods | Brief description	|
| ------------ | ------------------------------------------------------------ |
| onAwake	| Executed after the component is activated. At this time, all nodes and components have been created. This method will only be executed once |
| onEnable 	| Executed after the component is enabled, for example, executed after the node is added to the stage	|
| onStart  	| Executed before the first update is executed, it will only be executed once.	|
| onUpdate 	| Executed when each frame is updated, try not to write large loop logic here or use the getComponent method |
| onLateUpdate | Executed after each frame update, try not to write large loop logic here or use the getComponent method |
| onDisable	| Executed when the component is disabled, such as after the slave node is removed from the stage	|
| onDestroy	| Executed when manually calling node destruction	|
| onReset	| Reset component parameters to default values. If this function is rewritten, the component will be reset and automatically recycled to the object pool to facilitate reuse next time. If there is no reset, recycling and reuse will not be performed. |

In addition to the above, there are also the life cycle of physical events, the life cycle of click events, the life cycle of keyboard events, etc.

You can view the specific API

https://layaair2.ldc2.layabox.com/api2/Chinese/index.html?version=2.9.0beta&type=Core&category=Component&class=laya.components.Script

### 3. The difference between the use of Runtime and scripts

The core of LayaAir's component development is the reasonable use of Runtime classes and Script (script component) classes, and the use of life cycle methods.

Many developers don't quite understand the difference between the Runtime class and the script component class in the IDE.

The Runtime class is actually the UI inheritance class.

UI inheritance classes are divided into two major types, one is scene inheritance class, and the other is UI component inheritance class.

The scene inheritance class, because it inherits the entire scene, is more convenient for node search and node control of the entire scene. Therefore, the scene class usually plays the role of a manager, responsible for the global display control of the current scene, and does not need to do specific game logic. live.

As for the scene script, in terms of division of labor, try not to deal with display-related matters, and the display control is directly handed over to the scene inheritance class for coordination. The script of the scene is mainly responsible for logical control. Is the role of the logic controller. The specific work is done by node scripts. Each node script is responsible for the behavior control of its own node. In these node scripts, there is no need for coupling. Just take care of yourself and handle your own business logic. It is completely a single-function design idea.

Component inheritance classes are usually used for general component display effects. Different from the node scripts responsible for the real project business logic, the component inheritance class is an expansion and supplement to the capabilities of the UI component itself, and is a universal component display effect implementation. For example, when a button is clicked, it must all achieve a click-to-zoom effect. Then you can write a Runtime class that inherits the button, and implement click zoom in this class. When bound to the button's Runtime, all nodes bound to the inherited class will have this scaling effect. Although it is used for multiple nodes, it inherits from components and extends components. In essence, it enriches the functions of components and is separated from the specific business logic of the product. It also conforms to the ideas of decoupling and single responsibility.

Therefore, only by clearly understanding the functions of the engine can we achieve clear responsibilities.

##### in conclusion,

The Runtime class of the scene is responsible for global control of the display of the entire scene, such as the display of the global start and end, the change of global points, etc.

The script class of the scene is responsible for all the logical control in the scene. For example, when the player clicks start, the scene inheritance class listens for this click operation. It does not start the game immediately, but notifies the script class of the scene that the game has started and the scene Only the control script has the right to decide whether the game should start, under what logic, who will appear next, the position of the game, etc.

The script class of the node is responsible for the specific logic execution. Once the game characters and NPCs appear in the scene, specific attacks and injuries, etc., are handled separately and should not be piled into the scene class. For example, when a block falls due to gravity, when it rebounds and when it breaks, this is handled by the block's script based on its own collision feedback, and the scene control script does not need to care. Only in the block's logic script, if it determines that it has hit the floor, does it need to notify the scene control script: the game is over. The scene control script then notifies the scene inheritance class: the game is over, pop up the end panel.

Therefore, by clearly understanding their respective functions, clear division of labor, and single responsibility, decoupled component development can be achieved.

By making good use of LayaAir 2.0 engine component development, development efficiency and maintenance efficiency will be greatly improved, and the code structure will become clearer and easier to understand.

If you still don’t quite understand after reading this article,

You are also welcome to read more intuitive video teaching videos

##### Introduction to component-based development video:

https://ke.qq.com/course/427324

##### 2D componentization practical lesson video:

https://ke.qq.com/course/2646524






