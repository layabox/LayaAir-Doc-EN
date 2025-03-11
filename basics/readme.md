# LayaAir3.0 engine function overview

> Author：Charley

LayaAir3.0 engine includes three parts: engine code, project development tools, and project release. This article only gives a brief overview of most functions to give developers a preliminary understanding.



## 1. Engine code

Engine code, in addition to the open programmable rendering pipeline, full-platform graphics engine architecture, next-generation PBR rendering flow, ClusterLighting multi-light technology, Forward+ rendering pipeline, high-performance parallel renderer API access (WebGPU) and other core foundations . For developers, we will introduce them from three parts: general, 2D, and 3D.

### 1.1 General parts of the engine

- Network (HTTP request, WebSocket request)
- Loading (can load text, JSON, XML, binary, audio, video, skeleton files, image files and other resources)
- ECS component system (component system, life cycle method)
- Scene management
- Events (dispatching, listening, capturing)
- Interaction (mouse, keyboard, screen touch, VR controller)
- Multimedia playback (audio, video)
- Easing
- Browser interface (encapsulates commonly used browser window functions and calls browser window functions)
- Device interface (gyroscope, accelerometer, geolocation)
- node
- Screen adaptation
- Mini-game adaptation (WeChat mini-games, Douyin mini-games, OPPO, vivo, Xiaomi, etc.)

### 1.2 2D engine

- 2D sprites (2D basic display objects and containers)
- 2D view (Scene2D, Dialog)
- 2D animation (atlas animation, frame-by-frame animation, easing animation, timeline animation, dragon bone, spine bone)
- 2D text (text, HTML text, BitmapFont)
- 2D UI components (Image, Button, Label, TextInput, TextArea, Combobox, CheckBox, Radio, RadioGroup, Tab, ViewStack, Clip, FontClip, VScrollBar, HcrollBar, Progressbar, VSlider, HSlider, ColorPicker, Box, List, Tree, Panel)
- 2D UI effects (mask, filter)
- 2D scene inheritance class (runtime class)
- 2D drawing (drawing rectangles and rounded rectangles, circles and sectors, polygons, line segments, polylines, curves, textures and filled textures)
- 2D physics (Box2D)
- Tiled Map

### 1.3 3D engine

- 3D sprites (3D basic display objects and containers)
- 3D basic tools (3D coordinate system, 3D transformation, 3D math tools, etc.)
- 3D scenes (scene management, ambient light, ambient reflection, scene sky, scene fog, etc.)
- 3D camera
- 3D lighting (directional light, point light, spot light, area light, shadow, light effect)
- 3D mesh
- 3D materials (model materials, particle materials, trailing materials, sky materials)
- 3D textures
- 3D particle system
- 3D trailing
- 3D physics (bullet, PhysX)
- 3D animation (rigid body animation, material animation, skeletal animation, camera animation, timeline animation)
- Custom Shader
- webXR



## 2. IDE (Integrated Development Environment)

An overview of the IDE is also divided into general modules, 2D modules, and 3D modules for an overview.

### 2.1 Common module

- Hierarchy management panel
- Assets panel
- Scene window
- Game window
- Console panel
- Timeline Animation panel
- Animation controller panel
- Inspector panel
- Project settings panel
- IDE plug-in System
- IDE Assets store

### 2.1 2D module

- 2D layout widgets (basic display object nodes, UI components, skeletal animation nodes)
- 2D animation editing
- 2D UI editor
- 2D script management
- Scene inheritance class management
- 2D prefabs

### 2.2 3D module

- 3D scene editing
- 3D camera
- 3D lighting settings
- 3D animation editing
- 3D particle system
- 3D material editing
- 3D blueprint editing
- 3D prefabs
- 3D physics editor



## 3. Project release

### 3.1 Web version released

Publishing the web version is the basic publishing method, which can be used to run on the browser or native packaged publishing.

### 3.2 Mini game release

It provides adaptation libraries for each mini-game platform, as well as quick release functions for each mini-game platform.

### 3.3 Native packaging and release

Supports publishing as installation packages for iOS and Android platforms.

