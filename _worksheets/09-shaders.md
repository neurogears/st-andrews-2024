---
layout: worksheet
title: Shaders
permalink: /tutorials/shaders.html
local: "true"
---

Getting Started
---------------

1. Install **BonVision** from the package manager, make sure to select **Community Packages** for the package source dropdown. ![The Bonsai package manager]({{ site.baseurl }}/assets/images/packagemanager.png)
2. This should also install the **Bonsai.Shaders package** as a dependency, check it is installed in the **installed** tab.

### **Exercise 1:** Creating a shader window, loading resources

![Setting up a window]({{ site.baseurl }}/assets/images/shaders-setup.svg)

* Create a `CreateWindow` source.
* Insert `BonVisionResources`.
* Insert `ShaderResources`.
* Insert `LoadResources`.
* Run the workflow to check it creates a display window.
* Try altering the properties of the `CreateWindow` source (e.g. ClearColor).

### **Exercise 2:** Rendering a plane

![Rendering a plane]({{ site.baseurl }}/assets/images/shaders-basic-plane.svg)

* Create a `RenderFrame` source.
* Insert an `OrthographicView`.
* Create a subject for this branch with `PublishSubject` called 'DrawCall'.
* Set the boundaries of the `OrthographicView` to a -1:1 basis (bottom: -1, left: -1, top: 1, right: 1).
* Subscribe to the `DrawCall` subject.
* Insert a `CreateVector4`.
* Insert an `UpdateUniform`. Set the `ShaderName` to 'Color' and the `UniformName` to 'color'.
* Insert a `DrawMesh`. Set the `MeshName` to 'Plane' and the `ShaderName` to 'Color'.
* Run the workflow and check that the plane is being rendered in the display window.
* What happens when the `CreateVector4` parameters are changed?
* Look at the [Color shader](https://github.com/bonvision/BonVision/blob/main/BonVision/Shaders/Color.frag), can you explain its behaviour in response to changes in `CreateVector4`?

### **Exercise 3:** Creating a custom fragment shader

* Create a new shader file called `posColor.frag` and add the following code:

```
#version 400
in vec2 texCoord;
out vec4 fragColor;

void main() {
    fragColor = vec4(texCoord.x, texCoord.y, 0.0, 1.0);
}
```

* Edit `ShaderResources` (double-click) and add a new Material.
  * Add 'posColor' in the `Name` field.
  * Select the `posColor.frag` shader file created in the previous step as the `FragmentShader`.
  * We'll borrow the vertex shader from BonVision for now, paste `BonVision:Shaders.Quad.vert` in the `VertexShader` field.
  * Close the Shader configuration editor with OK.

![Custom shader plane]({{ site.baseurl }}/assets/images/shaders-posColor-plane.svg)

* Remove `CreateVector4` and `UpdateUniform` from the DrawCall branch.
* Change `ShaderName` on `DrawMesh` to `posColor`.
* Run the workflow and observe the color profile of the rendered quad. Can you explain the color gradient pattern?

![Scale quad]({{ site.baseurl }}/assets/images/shaders-scale-quad.svg)

* Add a `Scale` after the `DrawCall` subscription and set X: 2, Y: 2.
* Insert an `UpdateUniform`, set the `ShaderName` to `posColor` and the `UniformName` to transform. Which shader is this uniform targeting?

### **Exercise 4:** Dynamically updating shader properties

* Update the `posColor.frag` shader file to include a new `uniform double` property and use it as a parameter of the output `fragColor`:

```
#version 400
uniform double dVal;
in vec2 texCoord;
out vec4 fragColor;

void main() {
    fragColor = vec4(texCoord.x, texCoord.y, dVal, 1.0);
}
```

![Dynamic uniform update]({{ site.baseurl }}/assets/images/shaders-mouse-uniform.svg)

* In a new branch add a `MouseMove` source.
* Insert `NormalizedDeviceCoordinates` to normalize mouse position relative to the display window size, and expose the `X` coordinate output.
* Insert `Rescale` to remap the X coordinate between 0 and 1 (Max: 1, Min: -1, RangeMax: 1, RangeMin: 0).
* Insert an `UpdateUniform` with `ShaderName` as `posColor` and `UniformName` as `dVal`
