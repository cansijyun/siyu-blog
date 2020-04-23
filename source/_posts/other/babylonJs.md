---
title: BabylonJs
date: 19-03-03 20:19:47
tags:- 3d
categories: 其他
---

# Cameras

Of the many cameras available in Babylon.js the two most used are probably - the Universal Camera used for First Person Movement and the Arc Rotate Camera which is an orbital camera. Though with the advent of WebVR this may change.

For input control by the user all cameras need to be attached to the canvas once constructed using

```javascript
camera.attachControl(canvas, true);
```

The second parameter is optional and defaults to **false**. When **false** then default actions on a canvas event are prevented. Set to true to allow canvas default actions.

**Notes**

1. A [Gamepad](https://doc.babylonjs.com/How_To/how_to_use_gamepads) may be used a controller.
2. For touch control either [PEP](https://github.com/jquery/PEP) or [hand.js](https://github.com/Deltakosh/handjs) is needed.

## Universal Camera

This was introduced with version 2.3 of Babylon.js and is controlled by the keyboard, mouse, touch or [gamepad](https://doc.babylonjs.com/How_To/how_to_use_gamepads) depending on the input device used, with no need for the controller to be specified. This extends and replaces the [Free Camera](https://doc.babylonjs.com/api/classes/babylon.freecamera), the [Touch Camera](https://doc.babylonjs.com/api/classes/babylon.touchcamera) and the [Gamepad Camera](https://doc.babylonjs.com/api/classes/babylon.gamepadcamera) which are all still available.

The Universal Camera is now the default camera used by Babylon.js if nothing is specified, and it’s your best choice if you’d like to have a FPS-like control in your scene. All demos on babylonjs.com are based upon that feature. Plug a Xbox controller into your PC and using it you’ll still be able to navigate most of the demos.

The default actions are:

1. Keyboard - The left and right arrows move the camera left and right, and up and down arrows move it forwards and backwards;
2. Mouse - Rotates the camera about the axes with the camera as origin;
3. Touch - Swipe left and right to move camera left and right, and swipe up and down to move it forward and backwards;
4. [gamepad](https://doc.babylonjs.com/How_To/how_to_use_gamepads) - corresponds to device.

**Note**

- Using keys in the Playground requires you to click inside the rendering area to give it the focus.

### Constructing a Universal Camera

```javascript
// Parameters : name, position, scene
    var camera = new BABYLON.UniversalCamera("UniversalCamera", new BABYLON.Vector3(0, 0, -10), scene);

// Targets the camera to a particular position. In this case the scene origin
    camera.setTarget(BABYLON.Vector3.Zero());

// Attach the camera to the canvas
    camera.attachControl(canvas, true);
```

[A Playground Example of a Universal Camera](https://www.babylonjs-playground.com/#SRZRWV) - 



## Arc Rotate Camera

This camera always points towards a given target position and can be rotated around that target with the target as the center of rotation. It can be controlled with cursors and mouse, or with touch events.

Think of this camera as one orbiting its target position, or more imaginatively as a spy satellite orbiting the earth. Its position relative to the target (earth) can be set by three parameters, *alpha* (radians) the longitudinal rotation, *beta* (radians) the latitudinal rotation and *radius* the distance from the target position. Here is an illustration:

![arc rotate camera](E:%5C%E9%99%88%E6%80%9D%E8%BF%9C%5ClocalRepo%5Csiyu-blog%5Csource%5Cimages%5Cfront-end%5Ccamalphabeta.jpg)

Setting *beta* to 0 or PI can, for technical reasons, cause problems and in this situation *beta* is offset by 0.1 radians (about 0.6 degrees).

Both *alpha* and *beta* increase in a clockwise direction.

The position of the camera can also be set from a vector which will override any current value for *alpha*, *beta* and *radius*. This can be much easier than calculating the required angles.

Whether using the keyboard, mouse or touch swipes left right directions change *alpha* and up down directions change *beta*.

### Constructing an Arc Rotate Camera

```javascript
// Parameters: alpha, beta, radius, target position, scene
    var camera = new BABYLON.ArcRotateCamera("Camera", 0, 0, 10, new BABYLON.Vector3(0, 0, 0), scene);

// Positions the camera overwriting alpha, beta, radius
    camera.setPosition(new BABYLON.Vector3(0, 0, 20));

// This attaches the camera to the canvas
    camera.attachControl(canvas, true);
```

[A Playground Example of an Arc Rotate Camera](https://www.babylonjs-playground.com/#SRZRWV#1) - 



Panning with an ArcRotateCamera is also possible by using CTRL + MouseLeftClick, the default action. You can specify to use MouseRightClick instead, by setting *useCtrlForPanning* to false in the *attachControl* call :

```javascript
   camera.attachControl(canvas, noPreventDefault, useCtrlForPanning);
```

If required you can also totally deactivate panning by setting :

```javascript
   scene.activeCamera.panningSensibility = 0;
```

## FollowCamera

The Follow Camera does what it says on the tin. Give it a mesh as a target and from whatever position it is currently at it will move to a goal position from which to view the target. When the target moves so will the Follow Camera.

The initial position of the Follow Camera is set when it is created then the goal position is set with three parameters:

1. the distance from the target - camera.radius;
2. the height above the target - camera.heightOffset;
3. the angle in degrees around the target in the x y plane.

The speed with which the camera moves to a goal position is set through its acceleration (camera.cameraAcceleration) up to a maximum speed (camera.maxCameraSpeed).

### Constructing a Follow Camera

```javascript
// Parameters: name, position, scene
var camera = new BABYLON.FollowCamera("FollowCam", new BABYLON.Vector3(0, 10, -10), scene);

// The goal distance of camera from target
camera.radius = 30;

// The goal height of camera above local origin (centre) of target
camera.heightOffset = 10;

// The goal rotation of camera around local origin (centre) of target in x y plane
camera.rotationOffset = 0;

// Acceleration of camera in moving from current to goal position
camera.cameraAcceleration = 0.005

// The speed at which acceleration is halted
camera.maxCameraSpeed = 10

// This attaches the camera to the canvas
camera.attachControl(canvas, true);

// NOTE:: SET CAMERA TARGET AFTER THE TARGET'S CREATION AND NOTE CHANGE FROM BABYLONJS V 2.5
// targetMesh created here.
camera.target = targetMesh;   // version 2.4 and earlier
camera.lockedTarget = targetMesh; //version 2.5 onwards
```

[A Playground Example of a Follow Camera following a moving target](https://www.babylonjs-playground.com/#SRZRWV#6) - 



## AnaglyphCameras

These extend the use of the Universal and Arc Rotate Cameras for use with red and cyan 3D glasses. They use post-processing filtering techniques.

### Constructing Anaglyph Universal Camera

```javascript
// Parameters : name, position, eyeSpace, scene
var camera = new BABYLON.AnaglyphUniversalCamera("af_cam", new BABYLON.Vector3(0, 1, -15), 0.033, scene);
```

### Constructing Anaglyph ArcRotateCamera

```javascript
// Parameters : name, alpha, beta, radius, target, eyeSpace, scene
var camera = new BABYLON.AnaglyphArcRotateCamera("aar_cam", -Math.PI/2, Math.PI/4, 20, new BABYLON.Vector3.Zero(), 0.033, scene);
```

The *eyeSpace* parameter sets the amount of shift between the left eye view and the right eye view. Once you are wearing your 3D glasses, you might want to experiment with this float value.

You can learn all about anaglyphs by visiting a [Wikipedia page that explains it thoroughly](http://en.wikipedia.org/wiki/Anaglyph_3D).

## Device Orientation Camera

This is a camera specifically designed to react to device orientation events such as a modern mobile device being tilted forward or back and left or right.

### Constructing a Device Orientation Camera

```javascript
// Parameters : name, position, scene
   var camera = new BABYLON.DeviceOrientationCamera("DevOr_camera", new BABYLON.Vector3(0, 0, 0), scene);

    // Targets the camera to a particular position
    camera.setTarget(new BABYLON.Vector3(0, 0, -10));

    // Sets the sensitivity of the camera to movement and rotation
    camera.angularSensibility = 10;
    camera.moveSensibility = 10;

    // Attach the camera to the canvas
    camera.attachControl(canvas, true);
```

[A Playground Example of a Device Orientation Camera](https://www.babylonjs-playground.com/#SRZRWV#3) - 

for those with a correct device.



## Virtual Joysticks Camera

This is specifically designed to react to Virtual Joystick events. Virtual Joysticks are on-screen 2D graphics that are used to control the camera or other scene items.

### Requires

The third-party file [hand.js](http://handjs.codeplex.com/releases/view/119684).

### Read

[Virtual Joysticks David Rousset Blog](http://blogs.msdn.com/b/davrous/archive/2013/02/22/creating-an-universal-virtual-touch-joystick-working-for-all-touch-models-thanks-to-hand-js.aspx) on David's blog.

### Video

[Virtual Joysticks Camera demo in video](https://www.youtube.com/watch?v=53Piiy71lB0)

![Screenshot of the Virtual Joysticks Camera in action on Espilit](E:%5C%E9%99%88%E6%80%9D%E8%BF%9C%5ClocalRepo%5Csiyu-blog%5Csource%5Cimages%5Cfront-end%5CVJCBabylon.jpg)

### Complete sample

Here is a complete sample that loads the Espilit demo and switches the default camera to a virtual joysticks camera:

```javascript
document.addEventListener("DOMContentLoaded", startGame, false);
function startGame() {
  if (BABYLON.Engine.isSupported()) {
    var canvas = document.getElementById("renderCanvas");
    var engine = new BABYLON.Engine(canvas, true);

    BABYLON.SceneLoader.Load("Espilit/", "Espilit.babylon", engine, function (newScene) {

      var VJC = new BABYLON.VirtualJoysticksCamera("VJC", newScene.activeCamera.position, newScene);
      VJC.rotation = newScene.activeCamera.rotation;
      VJC.checkCollisions = newScene.activeCamera.checkCollisions;
      VJC.applyGravity = newScene.activeCamera.applyGravity;

      // Wait for textures and shaders to be ready
      newScene.executeWhenReady(function () {
        newScene.activeCamera = VJC;
        // Attach camera to canvas inputs
        newScene.activeCamera.attachControl(canvas);
        // Once the scene is loaded, just register a render loop to render it
        engine.runRenderLoop(function () {
          newScene.render();
        }),
      }),
    }, function (progress) {
    // To do: give progress feedback to user.
    }),
  }
}
```

If you switch back to another camera, don’t forget to call the dispose() function first. Indeed, the VirtualJoysticks are creating a 2D canvas on top of the 3D WebGL canvas to draw the joysticks with cyan and yellow circles. If you forget to call the dispose() function, the 2D canvas will remain, and will continue to use touch events input.

## VR Device Orientation Cameras

A new range of cameras. [A Playground Example of a VR Device Orientation Camera](https://www.babylonjs-playground.com/#SRZRWV#4) - 

for those with a correct device.



### Constructing the VR Device Orientation Free Camera

```javascript
//Parameters: name, position, scene, compensateDistortion, vrCameraMetrics
var camera = new BABYLON.VRDeviceOrientationFreeCamera ("Camera", new BABYLON.Vector3 (-6.7, 1.2, -1.3), scene);
```

### Constructing the VR Device Orientation Arc Rotate Camera

```javascript
//Parameters: name, alpha, beta, radius, target, scene, compensateDistortion, vrCameraMetrics
var camera = new BABYLON.VRDeviceOrientationArcRotateCamera ("Camera", Math.PI/2, Math.PI/4, 25, new BABYLON.Vector3 (0, 0, 0), scene);
```

### Constructing the VR Device Orientation Gamepad Camera

```javascript
//Parameters: name, position, scene, compensateDistortion, vrCameraMetrics
var camera = new BABYLON.VRDeviceOrientationGamepadCamera("Camera", new BABYLON.Vector3 (-10, 5, 14));
```

## WebVR Free Camera

The new virtual reality camera

```javascript
// Parameters : name, position, scene, webVROptions
    var camera = new BABYLON.WebVRFreeCamera("WVR", new BABYLON.Vector3(0, 1, -15), scene);
```

This camera deserves a page to itself so here it is [Using the WebVR Camera](https://doc.babylonjs.com/How_To/WebVR_Camera);

## FlyCamera

This camera imitates free movement in 3D space, think "a ghost in space." It comes with an option to gradually correct Roll, and also an option to mimic banked-turns.

Its defaults are:

1. Keyboard - The **A** and **D** keys move the camera left and right. The **W** and **S** keys move it forward and backward. The **E** and **Q** keys move it up and down.
2. Mouse - Rotates the camera about the Pitch and Yaw (X, Y) axes with the camera as origin. Holding the **right mouse-button** rotates the camera about the Roll (Z) axis with the camera as origin.

### Constructing a Fly Camera

```javascript
// Parameters: name, position, scene
var camera = new BABYLON.FlyCamera("FlyCamera", new BABYLON.Vector3(0, 5, -10), scene);

// Airplane like rotation, with faster roll correction and banked-turns.
// Default is 100. A higher number means slower correction.
camera.rollCorrect = 10;
// Default is false.
camera.bankedTurn = true;
// Defaults to 90° in radians in how far banking will roll the camera.
camera.bankedTurnLimit = Math.PI / 2;
// How much of the Yawing (turning) will affect the Rolling (banked-turn.)
// Less than 1 will reduce the Rolling, and more than 1 will increase it.
camera.bankedTurnMultiplier = 1;

// This attaches the camera to the canvas
camera.attachControl(canvas, true);
```

## Customizing inputs

The cameras rely upon user inputs to move the camera. If you are happy with the camera presets Babylon.js is giving you, just stick with it.

If you want to change user inputs based upon user preferences, customize one of the existing presets, or use your own input mechanisms. Those cameras have an input manager that is designed for those advanced scenarios. Read [customizing camera inputs](https://doc.babylonjs.com/How_To/Customizing_Camera_Inputs) to learn more about tweaking inputs on your cameras.



# **Material和Texture

### 基础用法

```typescript
        var myMaterial = new BABYLON.StandardMaterial("myMaterial",  this._scene);
        // myMaterial.diffuseTexture = new BABYLON.Texture("./assets/texture/earth/earth_diffuse.jpg", this._scene);
        //js only method
        // myMaterial.diffuseTexture.uScale = 1;
        // myMaterial.diffuseTexture.vScale = -1;
        var diffuseTexture = new BABYLON.Texture("./assets/texture/earth/8k_earth_daymap.jpg", this._scene);
        //贴图反了，调整uv贴图,1和-1多试几遍，翻转UV
//如果翻转一般两个-1就可以了
        diffuseTexture.uScale = -1;
        diffuseTexture.vScale = -1;
        myMaterial.diffuseTexture = diffuseTexture;
        // myMaterial.bumpTexture = new BABYLON.Texture("./assets/texture/earth/earth_bump.jpg", this._scene);
        //不兼容黑白bump，需要转化成normal bump map converter
        myMaterial.bumpTexture = new BABYLON.Texture("./assets/texture/earth/normal_earth_bump.png", this._scene);
        myMaterial.specularTexture = new BABYLON.Texture("./assets/texture/earth/earth_specular.jpg", this._scene);
```

### 贴图反转办法

### 1.旋转模型到正确位置（最佳）

### 2.uv铁路重新做

## ^混合模式

```typescript
        var cloudSphere=BABYLON.MeshBuilder.CreateSphere("cloudSphere", {diameter: 30.5, segments: 0}, this._scene);
        var cloudMaterial = new BABYLON.StandardMaterial("cloudMaterial",  this._scene);
        cloudMaterial.diffuseTexture = new BABYLON.Texture("./assets/texture/earth/earth_cloud.png", this._scene);
        //相加混合，相当于threejs的AdditiveBlending，就是把被色边透明，相乘是非黑色变透明
                // let cloudTexture = new THREE.TextureLoader().load('./dist/img/earth_cloud.png');
                // let material = new THREE.MeshPhongMaterial();
                // material.map = cloudTexture;
                // material.transparent = true;
                // material.opacity = 1;
                // material.blending = THREE.AdditiveBlending;
        cloudMaterial.alphaMode=BABYLON.Engine.ALPHA_ADD;
        cloudSphere.visibility = 0.9999;
        cloudMaterial.opacityTexture = cloudMaterial.diffuseTexture;
        cloudSphere.material = cloudMaterial;

```



# *Lights

Lights are used, as you would expect, to affect how meshes are seen, in terms of both illumination and colour. All meshes allow light to pass through them unless shadow generation is activated. The default number of lights allowed is four but this can be increased.

![Elements](E:%5C%E9%99%88%E6%80%9D%E8%BF%9C%5ClocalRepo%5Csiyu-blog%5Csource%5Cimages%5Cfront-end%5Ctestlight.jpg)

*A pretty sphere with multiple lights*

## Types of Lights

There are four types of lights that can be used with a range of lighting properties.

### The Point Light

A point light is a light defined by an unique point in world space. The light is emitted in every direction from this point. A good example of a point light is a standard light bulb.

```javascript
var light = new BABYLON.PointLight("pointLight", new BABYLON.Vector3(1, 10, 1), scene);
```

### The Directional Light

A directional light is defined by a direction (what a surprise!). The light is emitted from everywhere in the specified direction, and has an infinite range. An example of a directional light is when a distant planet is lit by the apparently parallel lines of light from its sun. Light in a downward direction will light the top of an object.

```javascript
var light = new BABYLON.DirectionalLight("DirectionalLight", new BABYLON.Vector3(0, -1, 0), scene);
```

### The Spot Light

A spot light is defined by a position, a direction, an angle, and an exponent. These values define a cone of light starting from the position, emitting toward the direction.

The angle, in radians, defines the size (field of illumination) of the spotlight's conical beam , and the exponent defines the speed of the decay of the light with distance (reach).

*A simple use of a spot light*

```javascript
var light = new BABYLON.SpotLight("spotLight", new BABYLON.Vector3(0, 30, -10), new BABYLON.Vector3(0, -1, 0), Math.PI / 3, 2, scene);
```

### The Hemispheric Light

A hemispheric light is an easy way to simulate an ambient environment light. A hemispheric light is defined by a direction, usually 'up' towards the sky. However it is by setting the color properties that the full effect is achieved.

```javascript
var light = new BABYLON.HemisphericLight("HemiLight", new BABYLON.Vector3(0, 1, 0), scene);
```

## Color Properties

There are three properties of lights that affect color. Two of these *diffuse* and *specular* apply to all four types of light, the third, *groundColor*, only applies to an Hemispheric Light.

1. Diffuse gives the basic color to an object;
2. Specular produces a highlight color on an object.

In these playgrounds see how the specular color (green) is combined with the diffuse color (red) to produce a yellow highlight.

- [Playground example of a point light](https://www.babylonjs-playground.com/#20OAV9) - 

  

- [Playground example of a directional light](https://www.babylonjs-playground.com/#20OAV9#1) - 

  

- [Playground example of a spot light](https://www.babylonjs-playground.com/#20OAV9#3) - 

  

- [Playground example of a hemispheric light](https://www.babylonjs-playground.com/#20OAV9#5) - 

  

For a hemispheric light the *groundColor* is the light in the opposite direction to the one specified during creation. You can think of the *diffuse* and *specular* light as coming from the centre of the object in the given direction and the *groundColor* light in the opposite direction.

- [Playground example of a hemispheric light on two spheres](https://www.babylonjs-playground.com/#20OAV9#6) - 

White hemispheric light with a black groundColor is a useful lighting method.

### Intersecting Lights Colors

- [Playground example of intersecting spot lights](https://www.babylonjs-playground.com/#20OAV9#9) - 

## Limitations

Babylon.js allows you to create and register as many lights as you choose, but know that a single StandardMaterial can only handle a defined number simultaneous lights (by default this value is equal to 4 which means the first four enabled lights of the scene's lights list). You can change this number with this code:

```javascript
var material = new BABYLON.StandardMaterial("mat", scene);
material.maxSimultaneousLights = 6;
```

But beware! Because with more dynamic lights, Babylon.js will generate bigger shaders which may not be compatible with low end devices like mobiles or small tablets. In this case, babylon.js will try to recompile shaders with less lights.

- [Playground example of 6 interacting point lights](https://www.babylonjs-playground.com/#IRVAX#0) - 

## On, Off or Dimmer

Every light can be switched off using

```javascript
light.setEnabled(false);
```

and switched on with

```javascript
light.setEnabled(true);
```

Want to dim or brighten the light? Then set the *intensity* property (default values is 1)

```javascript
light0.intensity = 0.5;
light1.intensity = 2.4;
```

For point and spot lights you can set how far the light reaches using the *range* property

```javascript
light.range = 100;
```

## Choosing Meshes to Light

when a light is created all current meshes will be lit by it. There are two ways to exclude some meshes from being lit. A mesh can be added to the *excludedMeshes* array or add the ones not to be excluded to the *includedOnlyMeshes* array. The number of meshes to be excluded can be one factor in deciding which method to use. In the following example two meshes are to be excluded from *light0* and twenty three from *light1*. Commenting out lines 26 and 27 in turn will show the individual effect.

- [Playground Example Excluding Lights](https://www.babylonjs-playground.com/#20OAV9#8) - 

## Lighting Normals

How lights react to a mesh depend on values set for each mesh vertex termed *normals*, shown in the picture below as arrows giving the direction of the lighting normals. The picture shows two planes and two lights. One light is a spot light, the other is a point light. The front face of each plane is the one you see when the *normals* are pointing towards you, the back face the opposite side.

![Elements](E:%5C%E9%99%88%E6%80%9D%E8%BF%9C%5ClocalRepo%5Csiyu-blog%5Csource%5Cimages%5Cfront-end%5Cnormals6.jpg)

*A blue back-faced plane and a blue front-faced plane, with a spot light and point light*

As you can see, the lights only affect the front face and not the back face.

## Lightmaps

Complex lighting can be computationally expensive to compute at runtime. To save on computation, lightmaps may be used to store calculated lighting in a texture which will be applied to a given mesh.

```javascript
var lightmap = new BABYLON.Texture("lightmap.png", scene);
var material = new BABYLON.StandardMaterial("material", scene);
material.lightmapTexture = lightmap;
```

Note: To use the texture as a shadowmap instead of lightmap, set the material.useLightmapAsShadowmap field to true.

The way that the scene lights are blended with the lightmap is based on the lightmapMode of the lights in the scene.

```javascript
light.lightmapMode = BABYLON.Light.LIGHTMAP_DEFAULT;
```

This will cause lightmap texture to be blended after the lighting from this light is applied.

```javascript
light.lightmapMode = BABYLON.Light.LIGHTMAP_SPECULAR;
```

This is the same as LIGHTMAP_DEFAULT except only the specular lighting and shadows from the light will be applied.

```javascript
light.lightmapMode = BABYLON.Light.LIGHTMAP_SHADOWSONLY;
```

This is the same as LIGHTMAP_DEFAULT except only the shadows cast from this light will be applied.

- [Playground Example](https://www.babylonjs-playground.com/#ULACCM#2) - 

## Projection Texture

In some cases it would be nice to define the diffuse color of the light (Diffuse gives the basic color to an object) from a texture instead of a constant color. Imagine that you are trying to simulate the light effects inside of a cathedral. The light going through the stained glasses will be projected on the ground. This is also true for the light coming from a projector or the light effects you can see in a disco.

In order to support this feature, you can rely on the `projectionTexture` property of the lights. This is only supported by the **SpotLight** so far.

```javascript
var spotLight = new BABYLON.SpotLight("spot02", new BABYLON.Vector3(30, 40, 30),
        new BABYLON.Vector3(-1, -2, -1), 1.1, 16, scene);
spotLight.projectionTexture = new BABYLON.Texture("textures/stainedGlass.png", scene);
```

- [Playground Example](https://www.babylonjs-playground.com/#CQNGRK) - 

In order to control the projection orientation and range, you can also rely on the following properties:

- `projectionTextureLightNear` : near range of the texture projection. If a plane is before the range in light space, there is no texture projection.
- `projectionTextureLightFar` : far range of the texture projection. If a plane is before the range in light space, there is no texture projection.
- `projectionTextureUpDirection` : helps defining the light space which is oriented towards the light direction and aligned with the up direction.

The projected information is multiplied against the normal light values to better fit in the Babylon JS lighting. It also only impact the diffuse value. So it might be necessary to change the specular color of the light to better fit with the scene.

## Next step

With the use of these powerful lights, your scene is likely really starting to 'shine'. And don't forget that you can animate light positions, directions, colors, and therefore create wonderful 'light shows'. We'll talk about that soon, or have fun discovering how to do it on your own. Maybe you could do light property settings inside the scene's render loop function. Its fun and beautiful!

Guess what! The next tutorial... is about animation! [Click this and let's go!](https://doc.babylonjs.com/babylon101/Animations)







# **Positioning, Rotating and Scaling

## 初始位置设置

```

```



## 初始角度设置

```typescript
        this.earthSphere = BABYLON.MeshBuilder.CreateSphere("sphere", {diameter: 30, segments: 32}, this._scene);
        //初始转角
        this.earthSphere.rotation.x = Math.PI ;

        //初始转角
        // var axis = new BABYLON.Vector3(1, 0, 0);
        // var axisLine = BABYLON.MeshBuilder.CreateLines("axis", {points:[axis.scale(-90), axis.scale(90)]}, this._scene);
        // axis.normalize();
        // //转180度 theta
        // var angle  = Math.PI;
        // // @ts-ignore
        // var quaternion = new BABYLON.Quaternion.RotationAxis(axis, angle );
        // this.earthSphere.rotationQuaternion = quaternion;


        //初始转角
        // var axis = new BABYLON.Vector3(1, 0, 0);
        // var axisLine = BABYLON.MeshBuilder.CreateLines("axis", {points:[axis.scale(-90), axis.scale(90)]}, this._scene);
        // var angle = Math.PI;
        // this.earthSphere.rotate(axis, angle, BABYLON.Space.WORLD);
        // this.earthSphere.rotate(axis, angle, BABYLON.Space.LOCAL);
```



## 转角动画

```typescript
var createScene = function () {
    var scene = new BABYLON.Scene(engine);
    var camera = new BABYLON.ArcRotateCamera("camera1", 0, 0, 0, new BABYLON.Vector3(5, 3, 0), scene);
    camera.setPosition(new BABYLON.Vector3(10.253, 5.82251, -9.45717));

    camera.attachControl(canvas, false);
    var light = new BABYLON.HemisphericLight("light", new BABYLON.Vector3(0, 1, 0), scene);

    var faceColors = [];
    faceColors[0] = BABYLON.Color3.Blue();
    faceColors[1] = BABYLON.Color3.Red();
    faceColors[2] = BABYLON.Color3.Green();
    faceColors[3] = BABYLON.Color3.White();
    faceColors[4] = BABYLON.Color3.Yellow();
    faceColors[5] = BABYLON.Color3.Black();

    var options = {
        faceColors: faceColors
    };

    var boxW = BABYLON.MeshBuilder.CreateBox("BoxW", options, scene);
    boxW.rotation.y = Math.PI / 4;
    boxW.position = new BABYLON.Vector3(2, 3, 4);
    //复制一个
    var boxL = boxW.createInstance("boxL");
    /***************************************************************/

    /*************************World Axis for Rotation**********************/
    var axis = new BABYLON.Vector3(1, 1, 1);
    //在箱子的基础位置上画向量
    var drawAxis = BABYLON.MeshBuilder.CreateLines("vector", {
        points: [
            boxW.position.add(axis.scale(-2)),
            boxW.position.add(axis.scale(2))
        ]
    }, scene);
    /***************************************************************/

    /**************Animation of Rotation**********/
    //每一帧的转角
    var angle = 0.02;
    scene.registerAfterRender(function () {
        boxW.rotate(axis, angle, BABYLON.Space.WORLD);
        //boxL.rotate(axis, angle, BABYLON.Space.LOCAL);
    });

    showAxis(8);

    return scene;
};
```

## 绕轴旋转

```typescript
        var moonSphere = BABYLON.MeshBuilder.CreateSphere("sphere", {diameter: 6, segments: 15}, this._scene);
        // moonSphere.position.x=24;
        // moonSphere.position.y=10;
        var moonStart =  new BABYLON.Vector3(36, 20, 0);
        // moonSphere.position = moonStart;
        var moonMaterial = new BABYLON.StandardMaterial("moonMaterial",  this._scene);
        moonMaterial.diffuseTexture = new BABYLON.Texture("./assets/texture/earth/2k_moon.jpg", this._scene);
        moonSphere.material = moonMaterial;

        var CoR_At = new BABYLON.Vector3(0, 0, 0);
        var axis = new BABYLON.Vector3(20, -36, 0);
        var axisLine = BABYLON.MeshBuilder.CreateLines("axisLine", {points:[CoR_At.add(axis.scale(-50)),CoR_At.add(axis.scale(50))]}, this._scene);

        /********************Parent at Pivot Position, Position Child*********/
        var moonPivot = new BABYLON.TransformNode("root");
        moonPivot.position = CoR_At;
        moonSphere.parent = moonPivot;
        moonSphere.position = moonStart;

        //var alpha = 0;
        this._scene.registerBeforeRender(()=>{
            // cloudSphere.rotation.y += 0.002;
            this.cloudSphere.addRotation(0, 0.0008, 0.0002);
            this.earthSphere.addRotation(0, 0.001, 0);
            // moonSphere.addRotation(0, 0.005, 0);
            //支点转动相当于公转
            moonPivot.rotate(axis, 0.006, BABYLON.Space.WORLD);
            //三角函数法
            // alpha+=0.005
            // moonSphere.position = new BABYLON.Vector3(5 * Math.sin(alpha), moon.parent.position.y, 5 * Math.cos(alpha));
        });
```

There are a variety of ways within Babylon.js to position, rotate and scale a mesh, from simple methods to the use of matrices. All of which depend on you knowing which [frame of reference](https://doc.babylonjs.com/resources/Frame_Of_Reference), either the **world axes** or the **local axes**, is being used.

Prior to the *MeshBuilder* method of creating a mesh the only way to produce a cuboid or ellipsoid, for example, was to create a cube and sphere and scale them in one dimension or another. This could produce difficulties with subsequent manipulations of a mesh. Since using *MeshBuilder* allows you to set different sizes for meshes in the x, y and z directions these difficulties with [scaling](https://doc.babylonjs.com/babylon101/position#scaling) no longer arise.

There are two types of tactic to position and rotate a mesh; one type is the **set-at** tactic and the other is **move-by**. [Position](https://doc.babylonjs.com/babylon101/position#position) and [rotation](https://doc.babylonjs.com/babylon101/position#rotation) are both of the **set at** type, the values being given set the actual position and rotation of the mesh. On the other hand [addRotation](https://doc.babylonjs.com/babylon101/position#sequencing-rotations) is a **move-by** type since it adds the given rotation around one axis to the current rotation of the mesh. You can read about more **set-at** and **move-by** types below.

## Set-At Methods

In addition to *position*, which places a mesh according to the **world axes**, and *rotation* which sets the orientation with [Euler angles](https://doc.babylonjs.com/resources/Rotation_Conventions), the following are available to set a mesh's position and rotation.

### Position using Local Axes

This *setPositionWithLocalVector* method sets the position with reference to **local space** which is a frame of reference that uses the origin of the **world axes** and axes parallel to those of the mesh's **local axes**.

```javascript
mesh.setPositionWithLocalVector(new BABYLON.Vector3(x, y, z));
```

- [Playground Example setPositionWithLocalVector](https://www.babylonjs-playground.com/#EYZE4Q#2) - 

In order to obtain the current position of the object in **local space** use

```javascript
var localPosition = mesh.getPositionExpressedInLocalSpace();
```

### RotationQuaternion

All angles are in radians

A [rotationQuaternion](https://doc.babylonjs.com/resources/Rotation_Conventions#quaternions) sets the orientation of a mesh by a rotation around a given axis. It is a four dimensional vector of the form (x, y, z, w). The most straight forward way to use a rotationQuaternion is as follows

```javascript
var axis = new BABYLON.Vector3(1, 1, 1);
var angle = Math.PI / 8;
var quaternion = new BABYLON.Quaternion.RotationAxis(axis, angle);
mesh.rotationQuaternion = quaternion;
```

The default for rotationQuaternion is *undefined* . When a *rotationQuaternion* is set the value of *rotation* is set to (0, 0, 0).

- [Playground Example rotationQuaternion](https://www.babylonjs-playground.com/#EYZE4Q#3) - 

**Note** :

1. You MUST set and use rotationQuaternion when creating physics objects because physics engines rely only on them.
2. Setting a `rotationQuaternion` overwrites the use of `rotation` see [warning](https://doc.babylonjs.com/resources/Rotation_Conventions#warning)

## Align Axes

When you want to rotate a camera or mesh so that it lines up with a set of given axes you can use the [RotationFromAxis](https://doc.babylonjs.com/How_To/rotate#how-to-generate-a-rotation-from-a-target-system) method to find the needed [Euler angles](https://doc.babylonjs.com/resources/Rotation_Conventions) to use with *rotation* as follows

```javascript
var orientation = BABYLON.Vector3.RotationFromAxis(axis1, axis2, axis3);
mesh.rotation = orientation;
```

where *axis1*, *axis2* and *axis3* are three left-handed orthogonal vectors and the mesh will be aligned with

- *axis1* as the x axis in its local system
- *axis2* as the y axis in its local system
- *axis3* as the z axis in its local system

Using this a plane can be made to follow a curve so it lies parallel or perpendicular to the curve as it does so. In the following example a set of points is used to generate and draw a curve and a [3D path](https://doc.babylonjs.com/How_To/How_to_use_Path3D) is created. Babylon.js provides the way to obtain the normal, tangent and binormal from the *Path3D* object at each of the points used to generate it. These form a set of orthogonal vectors, and depending on the order they are used, a plane can be made to follow and track the shape of the curve. All six orders are used in the example, the top one [0] has the plane tangential to the curve and the fourth one down [3] is perpendicular to the curve. Others can twist the plane at certain points.

- [Playground Animation - RotationFromAxis](https://www.babylonjs-playground.com/#1PX9G0) - 

## Move-By Methods

These methods add the given value (positive or negative) to the current position or orientation of the mesh. In the case of *position* and *rotation* and *rotationQuaternion*. This can be done by actually adding values using '+=' or `-=" to individual components or using vector addition. The following animated playgrounds show examples.

- [Playground Animation - Position](https://www.babylonjs-playground.com/#66EBY3) - 
- [Playground Animation - Rotation](https://www.babylonjs-playground.com/#1ST43U#47) - 
- [Playground Animation - Rotation Along Straight Horizontal Path](https://www.babylonjs-playground.com/#92EYG#13) - 
- [Playground Animation - rotationQuaternion](https://www.babylonjs-playground.com/#1ST43U#44) - 

For rotating the most straight forward **move-by** method is [addRotation](https://doc.babylonjs.com/babylon101/position#sequencing-rotations) which increments the orientation of a mesh about one of the **local axes**.

- [Playground Animation - addRotation](https://www.babylonjs-playground.com/#EYZE4Q#5) - 

The following playground shows you how to use *addRotation* to construct wheels.

- [Playground Example - Wheels](https://www.babylonjs-playground.com/#1PON40#12) -  Author [Jerome Bousquie](http://jerome.bousquie.fr/BJS/demos/)

The other ways below move by the values given in the parameters.

### Locally Translate

This method moves, or more strictly translates, a mesh using the **local axes**

```javascript
mesh.locallyTranslate(new BABYLON.Vector3(x, y, z));
```

- [Playground Example locallyTranslate](https://www.babylonjs-playground.com/#EYZE4Q#1) - 
- [Playground Animation locallyTranslate](https://www.babylonjs-playground.com/#EYZE4Q#4) - 

### Translate

Mathematically a translation is a single vector which gives the direction and distance that an object moves by. The *translate* method in BABYLON.js requires three parameters direction, distance and space.

The space parameter allows you to state whether *translate* uses the **world axes** or the **local axes** and takes the values BABYLON.Space.WORLD and BABYLON.Space.LOCAL respectively.

By allowing direction and distance to be given Babylon.js gives you the opportunity to easily move a mesh along one the the axes x, y, or z. This is because Babylon.js supplies three constant unit vectors along each axis; these are BABYLON.Axis.X, BABYLON.Axis.Y and BABYLON.Axis.Z. So moving a distance 2 along the y axis becomes

```javascript
mesh.translate(BABYLON.Axis.Y, 2, BABYLON.Space.WORLD);
```

for the **world axes** and

```javascript
mesh.translate(BABYLON.Axis.Y, 2, BABYLON.Space.LOCAL);
```

for the **local axes**.

It is also possible to supply other unit vectors and a distance.

```javascript
mesh.translate(new BABYLON.Vector3(-1, 3, -2).normalize(), 10, BABYLON.Space.LOCAL);
```

Alternatively, if you wish just to supply a vector giving the whole of the translation use a unit distance

```javascript
mesh.translate(new BABYLON.Vector3(-1, 3, -2), 1, BABYLON.Space.WORLD);
```

Generally the total distance moved given vector v and distance d as parameters will be |v|d, so using

```javascript
mesh.translate(new BABYLON.Vector3(-6, 3, -2), 5, BABYLON.Space.WORLD);
```

will move the mesh a distance of 35 units in the direction (-6, 3, -2) since |(-6, 3, -2)| = √49 = 7 and 7 * 5 = 35

### Rotate

To rotate a mesh an axis, angle and the space are needed. The center of rotation is the origin of the **local axes** and the axis is given as any vector(x, y, z) and passes through the center of rotation. In other words the mesh spins at its current position. Changing the position of the center of rotation can be done by using a parent or a [pivot](https://doc.babylonjs.com/How_To/Pivot).

The axes BABYLON.Axis.X, BABYLON.Axis.Y and BABYLON.Axis.Z may be used. The frame of reference for the axis can be the **world axes** or the **local axes**.

```javascript
pilot.rotate(BABYLON.Axis.Y, Math.PI / 2, BABYLON.Space.WORLD);

pilot.rotate(new BABYLON.Vector3(-1, 3, -10), 7 * Math.PI / 12, BABYLON.Space.LOCAL);
```

**Note:** `mesh.rotate()` generates a new [quaternion](https://doc.babylonjs.com/resources/Rotation_Conventions#quaternions) and then uses `mesh.rotationQuaternion` while `mesh.rotation` is set to (0, 0, 0).

- [Playground Animation - Rotate](https://www.babylonjs-playground.com/#66EBY3#3) - 

## Change of Origin

Should you wish to position, rotate or scale a mesh about a point other than its own *local origin* then this can be done either using [a parent or a pivot](https://doc.babylonjs.com/How_To/Pivot) or by [coordinate transformation](https://doc.babylonjs.com/How_To/transform_coordinates).

### Parent

Assigning a mesh a [parent](https://doc.babylonjs.com/How_To/parenting) changes the **world space** for its children. Any change in position, orientation or scale of the parent will be applied to its children. Setting the position, rotation or scaling of a child will be done using the **local space** of the parent as the child's **world space**.

```javascript
childMesh.parent = parentMesh;
```

**Note** : Parent-child hierarchies are evaluated on every frame. So any position, rotation and scaling transformations made to the parent prior to assigning it to children will also be applied to the children when the parent is assigned. It usually makes sense not to rotate or move a child until after you've assigned it to the parent.

### Pivot

Using a [pivot](https://doc.babylonjs.com/how_to/pivots) can be tricky and a number of methods are available. There is one method that makes it possible to change the local origin of a mesh so that positioning, rotation and scaling transformations are based on the pivot as the new origin. This is done using `setPivotMatrix` with the second parameter `false`

```javascript
mesh.setPivotMatrix(BABYLON.Matrix.Translation(x, y, z), false);
```

In the following playground having set the pivot point (shown by the red sphere) to the top, back, right corner (C) of the box then positioning the box places C at that position. (Note the use of the negatives in the translation).

- [Playground Example Change Local Origin to Pivot](https://www.babylonjs-playground.com/#3RTT8P#78) - 

### Transform Coordinates

This method allows you to [transform coordinates](https://doc.babylonjs.com/How_To/Transform_Coordinates) without assigning a parent or a pivot, though it is often more straight forward to do so.

The change from a local position to a global position is achieved using

```javascript
mesh.computeWorldMatrix();
var matrix = mesh.getWorldMatrix();
var global_position = BABYLON.Vector3.TransformCoordinates(local_position, matrix);
```

# Further Reading

## Basic - L1

[Positions, rotations, scaling 101](https://doc.babylonjs.com/babylon101/Position)

## More Advanced - L3

[How To Use Translations and Rotations](https://doc.babylonjs.com/How_To/Rotate)
[How To Set and Use a Pivot](https://doc.babylonjs.com/How_To/Pivots)
[How To Rotate Around an Axis About a Point](https://doc.babylonjs.com/How_To/Pivot)
[How To Use Path3D](https://doc.babylonjs.com/How_To/How_to_use_Path3D)
[How To Use a Parent](https://doc.babylonjs.com/How_To/Parenting)
[How To Transform Coordinates](https://doc.babylonjs.com/How_To/Transform_Coordinates)
[Euler Angles and Quaternions](https://doc.babylonjs.com/resources/Rotation_Conventions)
[Aligning Rotation to Target](https://doc.babylonjs.com/How_To/rotate#how-to-generate-a-rotation-from-a-target-system)
[Frame of Reference](https://doc.babylonjs.com/resources/Frame_Of_Reference)
[Baking Transformations](https://doc.babylonjs.com/resources/Baking_Transformations)
[In-Browser Mesh Simplification (Auto-LOD)](https://doc.babylonjs.com/How_To/In-Browser_Mesh_Simplification)

## Gamelet

[A Simple Car Following a Path