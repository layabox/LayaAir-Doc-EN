# Character Controller

> Author : Charley         Version \>= LayaAir 3.2

A **Character Controller** is a component in a game engine used to control the movement of players or NPCs (Non-Player Characters). It is typically used to simplify and optimize character movement and collision handling in 3D space, providing a dedicated interface to implement functions like character movement, jumping, and collision detection.

In essence, a character controller represents a constrained physical entity. Unlike a rigid body that completely follows Newton's laws of physics, a character controller uses a hybrid approach, combining deterministic character movement with interactions in the physical environment. It is neither a completely static collider nor a completely dynamic rigid body, but a special entity somewhere in between.

Unlike a static physics collider (Physics Collider), a character controller can move actively in the scene.

Unlike a dynamic rigid body (Rigidbody3D), the character controller's movement is not entirely controlled by the physics engine's force system but is achieved through explicit movement commands, which ensures that the game character's movement is more precise and controllable.

For the character controller, the engine defaults to using a **capsule collider shape (CapsuleColliderShape)** as the collision shape. This is because the capsule shape is best suited for simulating humanoid characters, as it can effectively handle collisions with the environment while maintaining stability.

The character controller has several built-in practical functions to handle common character movement scenarios. For example, the `stepHeight` property defines the maximum stair height the character can automatically climb, allowing them to smoothly walk up small steps without jumping. The `maxSlope` property sets the maximum angle of a slope the character can climb; any slope steeper than this angle will be considered un-climbable. The `move` and `jump` methods control the character's movement and jumping functions, respectively.

In game development, the character controller is particularly suitable for controlling the main character in first-person or third-person games. It solves problems that can arise from using a traditional 3D rigid body to control a character, such as sliding, tipping, or unstable collisions. With the character controller, developers can achieve precise character control, including movement, jumping, climbing, and interacting with the environment, while maintaining an appropriate sense of physical realism.

In the LayaAir3 engine, the class for the **character controller** is **CharacterController**. This is a physics component class that inherits from the PhysicsColliderComponent class, and it is an ideal choice for implementing high-quality character control systems.

-----

## 1\. Collision Shape Properties

A **collision shape** is a geometry used to describe how an object detects and responds to collisions with other objects. It defines the object's physical boundary, or its external shape, so that the physics engine can accurately determine if objects are in contact and calculate the post-collision reaction.

Because the character controller defaults to a capsule collision shape and it is not recommended to change it, you cannot change the collision shape type in the IDE. However, we provide properties for adjusting the collision shape's size (capsule radius, capsule height) and its offset (capsule offset) to fine-tune the collision shape.

### 1.1 Capsule Radius 

This property represents the radius of the capsule collision shape used by the character controller. It determines the size of the character's collision bounds in the horizontal direction. As shown in animated Figure 1-1:

![](img/1-1.gif) 

(Animated Figure 1-1)

### 1.2 Capsule Height 

This property represents the height of the capsule collision shape used by the character controller, determining the size of the character's collision bounds in the vertical direction. As shown in animated Figure 1-2:

![](img/1-2.gif) 

(Animated Figure 1-2)

### 1.3 Collision Shape Editing Tool

In addition to adjusting values in the properties panel, you can also modify the collision shape directly in the scene.

As shown in animated Figure 1-3, simply click "Show Collision Shape Editing Tool" at the top of the collision shape.

Once in editing mode, you will see several white cubes, which are the editing points. By dragging these editing points, you can easily change the shape's appearance.

![](img/1-3.gif) 

(Animated Figure 1-3)

### 1.4 Capsule Offset 

This property represents the center offset of the capsule collision shape used by the character controller, and it is used to control the position of the capsule collision shape within the character controller.

When the origin of the character model does not align with the actual collision center, you can use the center offset to adjust the capsule's position and ensure accurate collision detection.

![](img/1-4.gif) 

(Animated Figure 1-4)

## 2\. Collision Group Settings

Due to the character controller's unique features, many of the base collision class's properties are not commonly used and are therefore not displayed in the IDE. If you have a specific use case, you can enable them in the code. This section will only cover the commonly used collision group property from the base collision class.

### 2.1 Collision Group 

When you have complex collision requirements, for example, wanting to collide with some objects but not others, you need to use grouping and specify which collision groups can collide with each other.

The `collisionGroup` property is used to specify which collision group the current collider belongs to. In the IDE, you can click on `Edit Group` to go to the `Physics System` section of `Project Settings` to add collision groups and their names. As shown in Figure 2-1.

![](img/2-1.png) 

(Figure 2-1)

It's important to note that the name of the collision group is for easy identification in the IDE, but the engine's API uses values that are powers of 2.

For example, `Default` in Figure 2-1 is $2^0$, which is 1, while the custom `npc` group is $2^2$, with an actual value of 4. Following this pattern, the collision group with group ID 3 has a value of 8.

Therefore, if you set collision groups in the code instead of the IDE, the value for the engine's API must be a power of 2.

Here is a code example:

```typescript
// Use code to specify which collision group a collider belongs to
xxx.collisionGroup = 1 << 3; // The value is 2 to the power of 3 (8), which can be simply understood as group ID 3, making it easier to align with the IDE concept.
```

### 2.2 Can Collide With 

In the IDE, you can set the value for **Can Collide With** by selecting multiple group names, as shown in Figure 2-2. This is used to specify which groups of colliders the current object can collide with.

![](img/2-2.png)  

(Figure 2-2)

In addition to specifying custom collision groups, `Nothing` at the top means it won't collide with any group, while `Everything` means it can collide with any group.

If developers need to pass values through code, they can also use bitwise operations, as shown in the following example:

```typescript
// Specify that collider xxx can collide with a specific collision group
xxx.canCollideWith = 1 << 2; // Only collides with the group with ID 2 (value is 4).

// Specify that collider xxx can collide with multiple collision groups
xxx.canCollideWith = (1 << 1) | (1 << 2) | (1 << 5); // Only collides with groups 1, 2, and 5.

// Specify that collider xxx cannot collide with certain groups, but can with all others
xxx.canCollideWith = -1 ^ (1 << 3) ^ (1 << 6); // Does not collide with groups 3 and 6, but can collide with any other group.
```

-----

## 3\. IDE Panel Exclusive Properties

### 3.1 Gravity 

Gravity refers to a constant acceleration applied by the physics engine to a rigid body, typically simulating real-world gravitational pull.

Gravity affects a rigid body's motion, causing it to continuously accelerate in a certain direction (usually downwards, i.e., in the negative Y-axis direction), thereby simulating a realistic free-fall effect.

The default gravity value is **(0, -9.8, 0)**, meaning the object will fall downwards with an acceleration of 9.8 m/s², as shown in animated Figure 3-1.

![](img/3-1.gif)   

(Animated Figure 3-1)

### 3.2 Push Force 

When a character collides with other movable objects, the push force acts on the object, causing it to move.

As shown in animated Figure 3-2, with a push force set, the block obstacle does not stop the character. If the push force were 0, the character would be blocked and unable to push the block away.

![](img/3-2.gif) 

(Animated Figure 3-2)

### 3.3 Max Slope 

Max slope refers to the maximum angle of a slope that a character controller can climb. If the slope's angle exceeds this value, the character will be unable to move further up and may start to slide down or stop at the bottom of the slope.

As shown in animated Figure 3-3, when the max slope is set to 50, the character can easily walk up the cone, but it cannot pass the 90-degree block obstacle.

![](img/3-3.gif) 

(Animated Figure 3-3)

### 3.4 Step Height 

The character's step height is the maximum height of a step that the character can smoothly climb. If a step's height exceeds this set value, the character will not be able to cross it directly and will be blocked or need to find another way (e.g., go around it).

This value is usually between `0.1` and `1`. For example, in animated Figure 3-4, with a default max slope of 90 degrees and a step height of 0.5, the character can easily pass over the low obstacle but is blocked by the higher one.

![](img/3-4.gif) 

(Animated Figure 3-4)

-----

## 4\. In-Engine Movement Control

Character movement control must be implemented by calling methods in the code logic.

### 4.1 Character Position 

The character's position is used to get and set the position of the character controller. This is typically used to get the character's current position or set their position for initialization (e.g., spawn points or teleportation points).

Here is a code example:

```typescript
const { regClass, property } = Laya;
@regClass()
export default class DirectMove extends Laya.Script {
    declare owner: Laya.Sprite3D;
    private characterController: Laya.CharacterController;
    onAwake(): void {
        // Get the Character Controller component and assign it to characterController.
        this.characterController = this.owner.getComponent(Laya.CharacterController);
        // Set the spawn point position.
        this.characterController.position = new Laya.Vector3(0, 0, 0);
    }
}
```

### 4.2 Character Movement 

Character movement is used to move the character by specifying a movement vector.

In code, if you want to freely control the character's movement, this is typically done every frame in the `onUpdate()` method. By continuously calling the `move` method and passing a movement vector, the character will move in the specified direction and distance.

When you need to stop, you must not only stop calling the `move` method with the movement vector but also reset the movement vector to a zero vector. Otherwise, the character will continue to move based on the last set vector.

Here is a code example:

```typescript
const { regClass, property } = Laya;
@regClass()
export default class DirectMove extends Laya.Script {
    declare owner: Laya.Sprite3D;
    private characterController: Laya.CharacterController;
    // Flag to indicate if the character is moving.
    private ismoveing: boolean = false;
    // Movement vector: speed and direction. Move in the positive X-axis direction at a speed of 0.05.
    private moveVector: Laya.Vector3 = new Laya.Vector3(0.05, 0, 0);
    onAwake(): void {
        // Get the Character Controller component and assign it to characterController.
        this.characterController = this.owner.getComponent(Laya.CharacterController);
    }
    
    /**
     * Executed every frame during the update. Avoid writing large loop logic or using the getComponent method here.
     * This is a virtual method; just override it for use.
     */
    onUpdate(): void {
        // If the character can move (when ismoveing is true).
        if (this.ismoveing) {
            // Call the move method of the character controller to move the character according to the moveVector.
            this.characterController.move(this.moveVector);
        }
    }

    /**
     * Stops the character's movement.
     */
    public moveStop(): void {
        // Flag the character to stop moving, so it no longer updates every frame.
        this.ismoveing = false;
        // Reset the character's movement vector to a zero vector to stop its movement.
        this.characterController.move(Laya.Vector3.ZERO.clone());
    }
}
```

For a complete example of freely controlling character movement using the character controller, please create a `3D-RPG example` template project in LayaAirIDE version 3.2 or later and view the source code.

### 4.3 Character Jump 

Character jumping can control the character controller's jump direction and height. It is usually in the opposite direction of gravity (the positive Y-axis). The larger the absolute value, the higher the single jump.

Here is a code example:

```typescript
const { regClass, property } = Laya;
@regClass()
export default class DirectMove extends Laya.Script {
    declare owner: Laya.Sprite3D;
    private characterController: Laya.CharacterController;
	/** Jump vector: positive Y-axis direction, height 5. */
    private jumpVector: Laya.Vector3 = new Laya.Vector3(0, 5, 0);

    onAwake(): void {
        // Get the Character Controller component and assign it to characterController.
        this.characterController = this.owner.getComponent(Laya.CharacterController);
        // Set the spawn point position.
        this.characterController.position = new Laya.Vector3(0, 0, 0);
    }
    
    onKeyDown(evt: Laya.Event): void {
        switch (evt.keyCode) {
            case Laya.Keyboard.SPACE: // When the space key is pressed.
                this.characterController.jump(this.jumpVector); // Jump.
                break;
        }
    }
}
```

### 4.4 Get Vertical Velocity 

The vertical velocity method is primarily used to get the character's current velocity in the vertical direction (usually the Y-axis). This value reflects the character's state as it is affected by gravity, jumping, or other vertical movements.

Developers can check the vertical velocity to determine if the character is in the air or has just landed and implement corresponding logic based on the current state.

  - **Value is 0**: The character is on the ground.
  - **Positive value**: The character is ascending (e.g., jumping).
  - **Negative value**: The character is descending.

Here is a code example:

```typescript
const { regClass, property } = Laya;
@regClass()
export default class DirectMove extends Laya.Script {
    declare owner: Laya.Sprite3D;
    private characterController: Laya.CharacterController;

    onAwake(): void {
        // Get the Character Controller component and assign it to characterController.
        this.characterController = this.owner.getComponent(Laya.CharacterController);
    }

	onUpdate(): void {
        if (this.characterController.getVerticalVel() !== 0)
            // Output the character's vertical velocity to the console.
            console.log("Vertical Velocity", this.characterController.getVerticalVel());
	}
}
```


## 5\. Related Documentation

### ["physics3D"](../../../physicsEditor/physics3D/readme.md)