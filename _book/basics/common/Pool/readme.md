# Object pool usage



## 1. Overview

During the project development process, many objects are constantly created and removed, such as the creation and removal of character attack bullets, special effects, the destruction and refresh of NPCs (non-player characters), etc., which consumes a lot of performance during the creation process. , especially when the number is large, this will cause lag.

Therefore, using an object pool can avoid the creation of a large number of objects. If we put objects into the pool every time they are used, such as monsters, bullets, etc., and the monsters are killed and no longer needed, they can be placed in the pool. The next time they are used, they can be taken directly from the pool. It needs to be created only if the object pool does not exist.

The advantage of the object pool is that it reduces the overhead of instantiating objects, allows objects to be used repeatedly, and reduces the chance of new memory allocation and garbage collector running. In addition, when the object is removed, it is not immediately erased from the memory. Only when the memory is considered to be insufficient, the garbage collection mechanism will be used to clear it. Clearing is very memory intensive and may cause lags. Using the object pool will reduce the garbage objects of the program and effectively improve the running speed and stability of the program.

Next, let's take a look at how to use the object pool (Pool) in LayaAir.



## 2. Object Pool Pool

Pool is an object pool class, used for storage and reuse of objects. Reasonable use of object pools can effectively reduce the cost of object creation, avoid frequent garbage collection, and thus optimize game smoothness.

### 2.1 Obtain an object pool

```typescript
	/**
 	* Get the object pool based on the object type identification character.
 	* @param sign Object type identification character.
 	* @return object pool.
 	*/
	static getPoolBySign(sign: string): any[] {
    	return Pool._poolDic[sign] || (Pool._poolDic[sign] = []);
	}
```

Laya.Pool identifies and manages object pools through object type identifiers, which are string names. If this identifier does not exist in the object pool system, an object pool with the identifier will be created. Therefore, we can also define multiple object pools through identification to handle different types of objects respectively. For example, the attacking bullet is an object pool, and the NPC player is an object pool.

So, when we want to use an object pool, we need to call it in the code like this:

```typescript
let bulletPool = Laya.Pool.getPoolBySign("Bullet");
```

With the object pool, we can check the current situation of the object pool, such as checking the number of objects in the object pool, continuing to add objects, and so on. For example code:

```typescript
let bulletPool = Laya.Pool.getPoolBySign("Bullet");
// Check the number of objects in the current object pool
console.log( bulletPool.length );

if( bulletPool.length == 0 )
{
    //Put the bullet into the object pool
    pool.push( new Bullet() );
}    
```



### 2.2 Clean up an object pool

```typescript
	/**
 	* Clear the objects in the object pool.
 	* @param sign Object type identification character.
 	*/
	static clearBySign(sign: string): void {
    	if (Pool._poolDic[sign]) Pool._poolDic[sign].length = 0;
	}
```

For example, in a game, when a battle ends and there is no need for the object pool for bullets, we can clean up the object pool through code:

```typescript
Laya.Pool.clearBySign("Bullet");
```



### 2.3 Obtain objects from the pool

#### 2.3.1 Obtained by identification

```typescript
	/**
 	* Get an object of this type stored in the object pool based on the object type identification character passed in. If there is no object of this type in the object pool, null is returned.
 	* @param sign Object type identification character.
 	* @return An object of this type in the object pool. If there is no object of this type in the object pool, null is returned.
 	*/
	static getItem(sign: string): any {
    	var pool: any[] = Pool.getPoolBySign(sign);
    	var rst: any = pool.length ? pool.pop() : null;
    	if (rst) {
        	rst[Pool.POOLSIGN] = false;
    	}
    	return rst;
	}
```

This is the most basic operation. An example of getting an object from the object pool. If there is no available object in the object pool, null is returned. The usage code is as follows:

```typescript
let bullet = Pool.getItem("Bullet");
```

If the object obtained at this time is null, then we should consider there are two situations:

1. For an object that needs to be created very frequently, or the process of creating this object consumes performance, we can pre-create a group of objects during the loading process of this scene and put this group of objects into the object pool.

```typescript
//Create the object pool of bullets for the first time
let bulletPool = Laya.Pool.getPoolBySign("Bullet");
//Create 10 bullet objects and put them into the object pool
for( var i = 0 ; i < 10 ; i++ )
{
	//Create a bullet
	let bullet = new Bullet();
	bulletPool.push( bullet );
}
//When needed, you can get the bullet object from the object pool
let bullet = Pool.getItem("Bullet");
```

2. The performance requirements for creating objects are not high. We can create objects through the following methods and put the objects into the object pool at any time.

#### 2.3.2 Obtain through identification, create if not

```typescript
	/**
 	* <p>According to the incoming object type identification character, obtain an object instance of this type identification in the object pool. </p>
 	* <p>When there is no object identified by this type in the object pool, use the function passed in to create an object of this type to create a new object and return it. </p>
 	* @param sign Object type identification character.
 	* @param createFun Method used to create objects of this type.
 	* @param caller this object
 	* @return an object identified by this type.
 	*/
	static getItemByCreateFun(sign: string, createFun: Function, caller: any = null): any {
    	var pool: any[] = Pool.getPoolBySign(sign);
    	var rst: any = pool.length ? pool.pop() : createFun.call(caller);
    	rst[Pool.POOLSIGN] = false;
    	return rst;
	}
```

Based on the above situation, if there are no objects in the object pool, you can use this method to create objects at any time:

```typescript
let bullet = Laya.Pool.getItemByCreateFun("Bullet", function()
{
    //Create a bullet
	let bullet = new Bullet();
	// Get the object pool of bullets
	var pool = Laya.Pool.getPoolBySign("Bullet");
	//Put the bullet into the object pool, or not into the object pool, according to the developer's needs
	pool.push( bullet );
	// Return bullet object
	return bullet;
});
```



### 2.4 Recycle objects into the pool

#### 2.4.1 Recycling through objects

```typescript
	/**
 	* Put the object into the object pool corresponding to the type identifier.
 	* @param sign Object type identification character.
 	* @param item object.
 	*/
	static recover(sign: string, item: any): void {
    	if (item[Pool.POOLSIGN]) return;
    	item[Pool.POOLSIGN] = true;
    	Pool.getPoolBySign(sign).push(item);
	}
```

For example, during a battle in the game, when the bullet taken out of the object pool has ended its life cycle, we can recycle the bullet object into the object pool through code:

```typescript
Laya.Pool.recover("Bullet", bullet);
```



### 2.5 Obtain and recycle objects through class names

Often when we are doing complex system architecture, it is a good way to obtain and recycle objects by using class names. Laya's Pool object has already considered this situation for us, let's take a look first

#### 2.5.1 Obtaining objects through class names

```typescript
	/**
 	* <p>According to the incoming object type identification character, obtain an object instance of this type identification in the object pool. </p>
 	* <p>When there is no object identified by this type in the object pool, a new object is created and returned based on the type passed in. </p>
 	* @param sign Object type identification character.
 	* @param cls The class used to create objects of this type.
 	* @return an object identified by this type.
 	*/
	static getItemByClass<T>(sign: string, cls: new () => T): T {
    	if (!Pool._poolDic[sign]) return new cls();

    	var pool = Pool.getPoolBySign(sign);
    	if (pool.length) {
        	var rst = pool.pop();
        	rst[Pool.POOLSIGN] = false;
    	} else {
        	rst = new cls();
    	}
    	return rst;
	}
```

#### 2.5.2 Recycling based on class name

```typescript
	/**
 	* Recycle according to the class name. If the class has a class name, it will be recycled. If not, it will not be recycled.
 	* @param instance specific instance of the class
 	*/
	static recoverByClass(instance: any): void {
    	if (instance) {
        	var className: string = instance["__className"] || instance.constructor._$gid;
        	if (className) Pool.recover(className, instance);
    	}
	}
```

With these two corresponding methods, we don't need to care about the creation and recycling of each object in the code, we can only care about the internal logic of the object. For example, there are many skill special effects during combat. We can manage the object pool for each special effect in a unified way:

```typescript
export class EffectA {
    
	constructor() {
    	super();
	}
    
	static create(): Effect {
   	 Pool.getItemByClass(EffectA);
    }

	recover(): void {
   	 Pool.recoverByClass(this);
	}
    
}

export class EffectB {    
    
	constructor() {
    	super();
	}
    
	static create(): EffectB {
   	 Pool.getItemByClass(EffectB);
    }

	recover(): void {
   	 Pool.recoverByClass(this);
	}
    
}
```



In summary, the Pool provided by Laya is a relatively basic object pool. Developers can expand the use of the object pool according to their own needs, making it easier to implement more complex object management.

