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

![VR tracking]({{ site.baseurl }}/assets/images/bonvision-closedvr-tracking.svg)

* For this task, we want to be able to track an object in 3 dimensions:
  * X: Horizontal movement (i.e. left to right)
  * Y: Vertical movement (i.e. up and down)
  * Z: Forward movement (i.e. closed and further from the camera / screen)
* A convenient way to track the Z-axis is to have a tracked object with two same-colored markers at a fixed distance from each other. During tracking we can use the pixel-distance between these two markers as a proxy for distance.
  * In the camera view, as the object approaches the camera, the two markers will have a greater pixel distance between them.
* Set up a color tracking workflow that includes `BinaryRegionAnalysis` to track the object markers.
* Insert a `Condition` that filters the stream to include only events when both markers are present.
* Branch from the `Condition` and create 3 `PythonTransform` operators to calculate:
  * Z-axis (distance from camera)
  * X-axis (lateral position relative to camera)
  * Y-axis (vertical position relative to camera)
* Use `Rescale` to remap the outputs of these calculations. We want to convert pixel space to 'physical' distance in the 3D environment.
  * For example in the X-axis, if the camera image is 640 pixels wide, we would want to map 0:640 pixels to e.g. -2:2.
* Use `CombineLatest` to pack the rescaled outputs together.
* Use `ExpressionTransform` to convert the `Tuple` output to a dynamic class with named properties:

```
new (
  it.Item1 as Distance,
  it.Item2 as Lateral,
  it.Item3 as Vertical
)
```
* Use a `PublishSubject` called `Tracking` to publish this class.

![Visual environment]({{ site.baseurl }}/assets/images/bonvision-closedloop-visenz.svg)

* Set up the 3D visual environment.
* For `MeshResources` you can download 3D models from the [course repository](https://github.com/neurogears/st-andrews-2024/tree/bv-worksheet/assets/models).
* Double-click on `MeshResources` and add some downloaded models (Add >> TexturedModel).
* In a separate branch create a `RenderFrame` followed by a `PerspectiveView`. Create a `PublishSubject` to publish the `PerspectiveView` data (ViewMatrix, PerspectiveMatrix).
* In a separate branch (or multiple separate branches) use the `DrawModel` operator to populate the 3D scene. Use the `MeshName` property to select a model from `MeshResources`. 
* Experiment with the properties of each `DrawModel` operator to modify the position and appearance of objects in the scene.
* Experiment with using externalized properties of the `DrawModel` operators to dynamically modify these properties.

![Visual environment]({{ site.baseurl }}/assets/images/bonvision-closedloop-eyetransform.svg)

* Subscribe to the `Tracking` subject and use it to populate a `CreateVector3`.
  * Use an `InputMapping` to populate the vector, try and determine which tracked axes correspond to X, Y, Z of this translation vector.
* Externalize the `Eye` property of `PerspectiveView` which defines 'camera' position in the scene and connect the `CreateVector3`.
* Run the workflow and observe the results in the display window.

### **Challenge (optional):** Endless runner

* Create an 'endless runner' style game in Bonsai.

Example specs:
* Player constantly moves forward through space, can adjust lateral movement via camera object tracking.
* Obstacles spawn at random at some distance from the player and get closer as player moves forward, these objects must be avoided (or navigated towards e.g. score pickups).
  * How can you detect 'collisions'? Determining whether the player has interacted with or avoided an object.
* Player accumulates score over time (can you display this score on the display window?).
* If the player collides with an obstacle, the game resets.
