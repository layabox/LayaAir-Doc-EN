# Root Node and Popups

Author: Gu Zhu

In the UI system, we often need to pop up components that automatically disappear when the user clicks on empty space. This functionality is built into the UI system.

The APIs for showing and closing popups are provided in **GRoot**.

**GRoot** is the root node of the UI system and is automatically created when the game starts. There is only one instance in the entire game, and it is always displayed above all scenes. Generally, normal UI interfaces can be added directly to a scene (`Scene2D`) without adding them to GRoot. If you call `GRoot.showPopup` or display a window (`GWindow`), these components will automatically be added under GRoot.

Since GRoot has only one instance, you can access it via `GRoot.inst`.

* **showPopup** — Pops up a component. If a target is specified, the popup will adjust its position to appear below the target, creating a dropdown effect. You can also specify whether it should pop up above or below. The UI system automatically calculates the popup position based on the component size to ensure it doesn’t go off-screen. For example:

```TypeScript
// Pop up at the current mouse position
GRoot.inst.showPopup(aComponent);

// Pop up below aButton
GRoot.inst.showPopup(aComponent, aButton);

// Pop up at a custom position
GRoot.inst.showPopup(aComponent);
aComponent.pos(100, 100);
```

* Windows can also be shown via `showPopup`, which adds the feature of closing the window when clicking outside:

```TypeScript
let aWindow: GWindow;
GRoot.inst.showPopup(aWindow);

// The only difference from aWindow.show() is that clicking outside will close the window. Other usage remains the same.
```

* **hidePopup** — By default, clicking outside automatically closes the popup. You can also manually close it using this API. You can specify which popup to close, or if no argument is provided, all currently open popups will be closed.
