---
layout: worksheet
title: BonVision
permalink: /tutorials/bonvision.html
local: "true"
---

Getting Started
---------------

1. Install **BonVision** from the package manager, make sure to select **Community Packages** for the package source dropdown. ![The Bonsai package manager]({{ site.baseurl }}/assets/images/packagemanager.png)
2. This should also install the **Bonsai.Shaders package** as a dependency, check it is installed in the **installed** tab.

### **Exercise 1:** Rendering a 3D model

![Setting up a model render]({{ site.baseurl }}/assets/images/bonvision2-render-model.svg)

* Create a the display initialisation branch: `CreateWindow` --> `BonVisionResources` --> `MeshResources` --> `LoadResources`.
* Download a 3D model from the [course repository](https://github.com/neurogears/st-andrews-2024/tree/bv-worksheet/assets/models) e.g. `icosphere.obj`.
* Double-click `MeshResources` and add the downloaded model as a `TexturedModel`, give it a name like icosphere.
* In a new branch create a `RenderFrame` source and insert a `PerspectiveView` from BonVision. Publish the output as a `PublishSubject` called `DrawCall`.
* In a new branch, subscribe to `DrawCall` and insert a `DrawModel` from BonVision. In draw model select the imported 3D model in the `MeshName` property.
* When you the run the workflow you should see the rendered 3D object.
* In the workflow so far, what does the output of `PerspectiveView` represent? In which shader and to what property is this output applied? (HINT: Use right-click >> Go to definition... on the BonVision nodes to explore).
* Try changing the color of the rendered object using the albedo property.
* Try adjusting the scale, rotation and position of the rendered object.
* Try adjusting the properties of `PerspectiveView` - how do they affect the rendering of the object?
* Try adding other models to the rendered scene.

### **Exercise 2:** Animating scene objects

![Animating a model]({{ site.baseurl }}/assets/images/bonvision2-animate-model.svg)

* Externalize the `RotationY` property of `DrawModel` (right-click).
* Create a `RangeAnimation` operator. Set its duration to 3s, and have it move between 0-359 (`RangeBegin` : `RangeEnd`).
* Add a `Repeat` to cycle this animation infinitely. 
* Insert a `DegreeToRadian` from the `Numerics` package and hook the output to the `RotationY` property.
* The object should now spin on the y-axis.
* Try and implement other animations. E.g. can you make the object bob up and down? (HINT: Use the `Sin` operator from the `Numerics` package).

### **Exercise 3:** Benchmarking screen round-trip latency

In a previous worksheet (closed-loop) we measure the closed-loop latency of our Arduino serial port with a digital-feedback test.
Here we will adapt this method to estimate the closed-loop latency of our display system.

![Init closed loop]({{ site.baseurl }}/assets/images/bonvision-closedloop-init.svg)

* Insert a `CreateWindow`, `BonVisionResources`, `LoadResources` in a branch.
* In a new branch create a `BehaviorSubject` to store a color value (call it something like `CurColor`), initialised with `CreateVector4` with all properties as 1 to start with a white color.

![Init closed loop]({{ site.baseurl }}/assets/images/bonvision-closedloop-renderpath.svg)

* In a separate branch add a `RenderFrame`.
* We only need a 2D environment for this benchmark so we'll insert a `NormalizedView`.
* Every render call, we want to draw a 2D plane with the most recent set color. Use a `WithLatestFrom` that has a subscription to `CurColor` as the 2nd input.
* Can you explain why we use a `BehaviorSubject` in this context?
* Output the color item from `WithLatestFrom` and use it to update color property of the `Color` shader with `UpdateUniform`.
* Finally, insert `DrawMesh` with the `Plane` mesh and `Color` shader.

![Closing the loop]({{ site.baseurl }}/assets/images/bonvision-closedloop-closingloop.svg)

To close the loop, we want to use a photodetector (in this case the analog grayscale sensor) to switch the current color to a contrasting color whenever it detects a change: e.g. detect white --> change to black --> detect black --> change to white ...

* Shown above is a partially completed part of the workflow to achieve this. 
* `AnalogInput` is used to sample an analog port with a grayscale sensor connected.
* `LessThan` and `DistinctUntilChanged` are used to produce boolean events whenever the light level crosses a particular threshold.
* Complete this part of the workflow to produce a feedback switching behavior (HINT: refer back to Ex. 1 of the closed loop worksheet).
* Once this is working, how can you extend/modify this workflow to estimate the closed-loop latency?

* **Optional:** Can you create a workflow that measures and estimates input latency? Input latency is the time from a user-input (e.g. key press, button push) until a change appears on screen in response (e.g. color change).

### **Exercise 4:** Closed-loop VR

In this exercise, we will build a 3D scene that reacts to a tracked 'real-world' object. By moving the scene view relative to this object we'll turn the monitor into a simple augmented reality window. To begin we'll set up the object tracking.

