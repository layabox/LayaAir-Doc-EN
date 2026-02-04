# List (GList)
Author: Guzhu

<img src="img/1-1.png" alt="1-1" style="zoom:60%;" />

- `Template Node` Item node template. Drag a node from the hierarchy panel. This node must be a child of the GList node.
- `Init Item Num` Initial item quantity. If greater than 0, the specified quantity of items will be automatically created.
- `Is Demo` If checked, `Init Item Num` will only take effect in the IDE, not at runtime. That is, only as a demonstration use within the IDE.
- `Item Data` Can provide simple data for each item.
- `Layout` Refer to [Layout Container](../layout/readme.md)
- `Clipping` Whether to enable clipping. After enabling, content exceeding container dimensions will be hidden.
- `Selection` Refer to [Selection Support](../selection/readme.md)
- `Scoller` Refer to [Scroll Support](../scroller/readme.md)

### 1. Managing List Content

At runtime, you can directly modify the list's children through APIs like addChild/removeChild. However, in practical applications, list content is usually updated frequently. A typical usage is when receiving backend data, clearing the list, then re-adding all items. If creating and destroying UI objects each time, it will consume a lot of CPU and memory. Therefore, GList has a built-in object pool.

Display list management methods using the object pool:

- `addItemFromPool` Get an object from the pool (if available) or create a new one and add it to the list. If no parameters are used, the list's "Item Resource" settings are used; you can also specify a URL to create a specific object.
- `getFromPool` Get an object from the pool (if available) or create a new one.
- `returnToPool` Return an object to the pool.
- `removeChildToPool` Delete an item and return the object to the pool.
- `removeChildToPoolAt` Delete an item at a specified position and return the object to the pool.
- `removeChildrenToPool` Delete a range of items, or delete all, and return all deleted objects to the pool

Smart you should know that addItemFromPool = getFromPool + addChild, removeChildToPool = removeChild + returnToPool.

When applying the pool, we should be very careful. A continuously growing pool would be a disaster for the game, but not using the pool also affects game performance.

Here are examples of several incorrect usages:

Incorrect Example 1:

```TypeScript
aList.addChild(obj);
aList.RemoveChildrenToPool();
```

When adding objects, the pool isn't used, but when finally clearing the list, they're put into the pool. If this code runs continuously, the object pool will continue to grow, possibly causing memory overflow.

Correct approach: Should create objects from the pool. Change addChild to addItemFromPool.

Incorrect Example 2:

```TypeScript
for(let i=0;i<10;i++)
    aList.addItemFromPool();

aList.removeChildren();
```

Here 10 items are added, but when removing, their references aren't saved, nor are they returned to the pool, causing memory leak. Change aList.removeChildren to aList.removeChildrenToPool();

**Removing and destroying are two different things.** When you remove an item from the list, if it won't be used in the future, it should be destroyed; if it's still needed, please save its reference. **But if put into the pool, do not destroy the item again.**

When adding a large number of items, besides using loop methods like addChild or addItemFromPool, you can also use another callback method. First define a callback function for the list, for example:

```TypeScript
function renderListItem(index:number, obj:GButton) {
    obj.title = "" + index;
}
```

If using an object pool, this callback function may be called repeatedly for the same object, so be very careful when registering event listeners in the callback function. Avoid using temporary functions to prevent duplicate additions.

Then set this function as the list's rendering function:

```TypeScript
aList.itemRenderer = renderListItem;
```

Finally, directly set the total number of items in the list. This way, the list will adjust the current list container's object quantity, then call the callback function to render items.

```TypeScript
//Create 100 objects. Note: numChildren cannot be used here. numChildren is read-only.
aList.numItems = 100;
```

If the newly set item count is less than the current item count, the excess items will be returned to the pool.

When using this method to generate a list, if you need to update a certain item, you can call renderListItem(index, getChildAt(index)) yourself.

If you want to listen for the click event of a certain item, you don't need to add a Click event listener to each item. Instead, directly listen to the list's ClickItem event:

```TypeScript
list.on(Laya.UIEvent.ClickItem, this, this.onClickItem);

// The first parameter of the callback function is the currently clicked object
function onClickItem(item: GObject): void {
    console.log("Clicked object: " + item.title);

    // How to get the index of this object in the list
    let childIndex = list.getChildIndex(item);
}
```

From the code above, you can see that in the event callback, you can conveniently get the currently clicked object. If you need to get the index, you can use GetChildIndex. Note that the item type must be a button, i.e., GButton, to trigger the ClickItem event.

### 2. Virtual List

If the list has a very large number of items, for example hundreds or thousands, creating entity display objects for each item will consume a lot of time and resources. This UI system has a built-in virtual mechanism for lists, which means it only creates entity objects for items within the display range, and implements large-capacity lists by dynamically setting data.

There are several conditions for enabling a virtual list:

- Need to define itemRenderer.
- Need to create Scroller. Lists without Scroller cannot enable virtualization.

After meeting the conditions, you can enable the list's virtual function:

```TypeScript
aList.setVirtual();
```

**Tip: The virtual function can only be enabled, not disabled.**

The performance of a virtual list is closely related to the processing logic of itemRenderer. You should try to simplify the logic here. Operations like Promise, IO, and high-density calculations should not appear here, otherwise lag will occur. If you need to initiate asynchronous operations in itemRenderer, do not let the asynchronous operation save the ITEM instance and directly modify the ITEM instance in the callback. The correct approach is to let the asynchronous operation save the ITEM's index. After the asynchronous operation is completed, query whether the ITEM at this index has a corresponding display object. If yes, update it; if not, give up the update.

In addition, itemRenderer should not have operations like new that will generate GC, because during the scrolling process, itemRenderer will be called very frequently.

In a virtual list, ITEMS are reused. When an ITEM needs to be refreshed, itemRenderer will be called. You don't need to care about the timing of this call, nor can you depend on this timing.

In a virtual list, the quantity and order of display objects and items are inconsistent. The quantity of items can be obtained through numItems, while the quantity of display objects can be obtained through the component's API numChildren.

In a virtual list, you need to pay attention to the distinction between item index and display object index. The value obtained through selectedIndex is the item's index, not the display object's index. APIs like AddSelection/RemoveSelection also require the item's index. The conversion between item index and object index can be completed through the following two methods:

```TypeScript
//Convert item index to display object index.
let childIndex = aList.itemIndexToChildIndex(1);

//Convert display object index to item index.
let itemIndex = aList.childIndexToItemIndex(1);
```

When using a virtual list, we rarely need to access off-screen objects. If you really need to get the display object of an item at a specified index in the list, for example the 500th one, because this item is not currently in the viewport, for a virtual list, objects not in the viewport do not have corresponding display objects. So you need to first make the list scroll to the target position. For example:

```TypeScript
//Note here, because we need to immediately access the object at the new scroll position, the second parameter scrollItToView cannot be true, i.e., do not use animation effect
aList.scrollToView(500);

//Convert to display object index
let index = aList.itemIndexToChildIndex(500);

//This is the 500th object you want
let obj = aList.getChildAt(index);
```

The essence of a virtual list is the separation of data and rendering. People often ask how to delete or modify virtual list items. The answer is to first modify your data, then refresh the list. You don't need to get a certain item object to handle it.

There are two ways to refresh a virtual list:

- Use numItems to reset the quantity.
- GList.refreshVirtualList.

**Using addChild or removeChild to add or delete objects to a virtual list is not allowed. If you want to clear the list, you must set numItems=0, not removeChildren.**

Virtual lists support variable-sized items. You can dynamically change the item size in two ways:

- Use width, height, or size inside itemRenderer to change the item's size.
- The item establishes a linkage to internal components, then modify the content in itemRenderer to trigger the change of internal components, thereby automatically changing the item height. For example, if the item establishes a high-height linkage to an internal variable-height text, then when the text changes, the item height changes automatically.

**Except for these two methods, you cannot change the item size through other methods outside of itemRenderer, otherwise the virtual list arrangement will be disordered. But you can force trigger itemRenderer by calling refreshVirtualList.**

Virtual lists support mixing different types of items. First define a callback function for the list, for example

```TypeScript
//Return different resource URL strings according to different indexes
function getListItemResource(index:number) {
    let msg = _messages[index];
    if (msg.fromMe)
        return "url1.lh";
    else
        return "url2.lh";
}
```

Then set this function as the list's item provider:

```TypeScript
aList.itemProvider = getListItemResource;
```

For horizontally flowing, vertically flowing, and paged lists, unlike non-virtual lists with flow characteristics, the number of items per row or column in a virtual list is fixed. The list will create a default item during initialization to measure this quantity.

If you still need layout with unequal numbers of items per row or column, and must use virtualization, you can insert some empty components or empty graphics for placeholder, and set their width according to actual needs, thereby achieving that layout effect.

### 3. Circular List

A circular list is a list where the head and tail are connected. A circular list must be a virtual list. The method to enable a circular list is:

```TypeScript
    aList.setVirtualAndLoop();
```

Circular lists only support single-row or single-column layouts, and do not support flow layouts and paged layouts.

Because a circular list is connected head-to-tail, specifying an item index may appear in different positions. So when you need to specify a scroll position, try to avoid using the item index. For example, if you need the circular list to scroll left/up one grid or right/down one grid, the best method is to call the Scroller's API: scrollLeft/scrollRight/scrollUp/scrollDown.

### 4. List Item Runtime

The list can dynamically set the item's Runtime type through code, thereby implementing logic encapsulation for item display objects. For example

```TypeScript
//Assume there is a custom component class MyItem. Note that the base class must match the prefab root node type, generally GButton.
class MyItem extends Laya.GButton {
    //Note! You can only get child objects in onConstruct. It's recommended to do initialization here, not in the constructor
    onConstruct() {
        //this.xx = this.getChild("xx");
        //this.xx.on(Laya.Event.CLICK, this, this.onClick);
    }

    sayHello() {
        console.log("Hello from MyItem");
    }
}

//Set the list's item Runtime to MyItem
aList.itemPool.defaultRuntime = MyItem;
//After setting the item's Runtime, itemRenderer can directly access MyItem's methods and properties
aList.itemRenderer = (index: number, item: MyItem) => {
    item.sayHello();
};
```

This mechanism is very useful for virtual lists and can make itemRenderer code more concise and efficient.
