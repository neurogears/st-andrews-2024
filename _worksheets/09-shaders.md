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

### **Exercise 1:** Creating a display window, loading resources

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

**Note:** a `uniform` is another type of shader variable declaration similar to `in` and `out`. A `uniform` variable has the distinction that it [remains constant across invocation of all shader stages](https://www.khronos.org/opengl/wiki/Uniform_(GLSL)), while `in` and `out` may vary at different shader stages.
{: .notice--info}

![Dynamic uniform update]({{ site.baseurl }}/assets/images/shaders-mouse-uniform.svg)

* In a new branch add a `MouseMove` source.
* Insert `NormalizedDeviceCoordinates` to normalize mouse position relative to the display window size, and expose the `X` coordinate output.
* Insert `Rescale` to remap the X coordinate between 0 and 1 (Max: 1, Min: -1, RangeMax: 1, RangeMin: 0).
* Insert an `UpdateUniform` with `ShaderName` as `posColor` and `UniformName` as `dVal`

### **Exercise 5:** Working with time

Aside from spatial information, we may also want our shader to be dynamic in time.

* Update the `posColor.frag` shader to file to include a new `uniform float` to represent time.
* Change the `fragColor` output to vary with time in one of the color channels:

```
#version 400
uniform double dVal;
uniform float time;
in vec2 texCoord;
out vec4 fragColor;

void main() {
    fragColor = vec4(texCoord.x, sin(time), dVal, 1.0);
}
```

![Dynamic time update]({{ site.baseurl }}/assets/images/shaders-time-input.svg)

* Update the Bonsai workflow to `Accumulate` the elapsed time of `RenderFrame`. What does this accumulation represent?
* Convert the output to a float with `ExpressionTransform`.
    * `Convert.ToSingle(it)`
* Use the result to update the `time` property of the shader with `UpdateUniform`

You should now have a fragment shader that adjusts its color with:
1. Spatial position of the pixel on screen
2. Time since the start of display
3. User input

* Optional: experiment with using other [GLSL core functions](https://registry.khronos.org/OpenGL-Refpages/gl4/index.php) to produce different shader behaviors based on this input data.

### **Exercise 6:** Audio visualizer

* Create a new shader file `fractalPyramid.frag`:

**Note:** This exercise uses a shader adapted from an [existing ShaderToy example](https://www.shadertoy.com/view/tsXBzS). You can optionally use a different shader from this site and adapt it yourself.
  {: .notice--info}

```
#version 400
in vec2 texCoord;
out vec4 fragColor;

uniform float iTime;
uniform vec2 offset = vec2(0.5, 0.5);
uniform float SCALE = 100;
uniform float mod = 1.0;

vec3 palette(float d){
	return mix(vec3(0.2,0.7,0.9),vec3(1.,0.,1.),d);
}

vec2 rotate(vec2 p,float a){
	float c = cos(a);
    float s = sin(a);
    return p*mat2(c,s,-s,c);
}

float map(vec3 p){
    for( int i = 0; i<8; ++i){
        float t = iTime*0.2;
        p.xz =rotate(p.xz,t);
        p.xy =rotate(p.xy,t*1.89);
        p.xz = abs(p.xz);
        p.xz-=.5;
	}
	return dot(sign(p),p)/5.;
}

vec4 rm (vec3 ro, vec3 rd){
    float t = 0.;
    vec3 col = vec3(0.);
    float d;
    for(float i =0.; i<64.; i++){
		vec3 p = ro + rd*t;
        d = map(p)*.5;
        if(d<0.02){
            break;
        }
        if(d>100.){
        	break;
        }
        //col+=vec3(0.6,0.8,0.8)/(400.*(d));
        col+=palette(length(p)*.1)/(400.*(d));
        t+=d;
    }
    return vec4(col,1./(d*100.));
}

void main() {
    vec2 uv = (texCoord - offset) * SCALE * (1.0/mod);
    vec3 ro = vec3(0.,0.,-50.);
    ro.xz = rotate(ro.xz,iTime);

    vec3 cf = normalize(-ro);
    vec3 cs = normalize(cross(cf,vec3(0.,1.,0.)));
    vec3 cu = normalize(cross(cf,cs));

    vec3 uuv = ro+cf*3. + uv.x*cs + uv.y*cu;
    
    vec3 rd = normalize(uuv-ro);
    
    vec4 col = rm(ro,rd);

    fragColor = col;
}

//Adapted from https://www.shadertoy.com/view/tsXBzS
```

![Time step initialisation]({{ site.baseurl }}/assets/images/shaders-audio-setup.svg)

* Set up a display window and resource loading as before.
* Create a new material in the `ShaderResources`. Use `fractalPyramid.frag` as the `FragmentShader` and use the same vertex shader as before `BonVision:Shaders.Quad.vert`.
* Expose the `TimeStep.ElapsedTime` property from `RenderFrame` and use `Accumulate` to accumulate the value.
* Convert the data type of the `Accumulate` output to a float with an `ExpressionTransform`
  * `Convert.ToSingle(it)`
* Use the output of `ExpressionTransform` to `UpdateUniform` of the `iTime` uniform in the `fractalPyramid` shader.

![Audio visualizer]({{ site.baseurl }}/assets/images/shaders-audio-viz.svg)

* To get an audio feedback value we'll construct a branch that acts as an audio amplitude filter.
* Create an `AudioCapture` source, then a `Pow` operator with the power set to 2. This will give us an absolute amplitude from the audio buffer.
* Use `Average` to average the buffer amplitude to a single value, then expose the `Val0` output property.
* Use another `Pow` to rescale the output (e.g. 0.5) and use a `PublishSubject` called e.g. `AudioLevel`
* In a different branch, subscribe to `DrawCall` again and `Scale` the plane as before.
* Use `WithLatestFrom` to pair the outputs with the latest `AudioLevel` value, using an `ExpressionTransform` to convert the type to a float.
* Use the audio value output of `WithLatestFrom` to update the `mod` property of the `fractalPyramid` shader.
* The rendered plane should now react to both time and audio level acting as a simple audio visualizer.

### **Exercise 7:** Using textures

Instead of setting pixel color programatically, we can sample pixel colors from a pre-defined texture.

* Create a new fragment shader called `textureShader.frag`:

```
#version 400
uniform sampler2D tex;
in vec2 texCoord;
out vec4 fragColor;

void main() {
    vec4 texel = texture(tex, texCoord);

    fragColor = texel;
}
```

* We are using a new property type [`sampler2D`](https://www.khronos.org/opengl/wiki/Sampler_(GLSL)) to represent our input texture here, as well as another built-in GLSL function [`texture`](https://registry.khronos.org/OpenGL-Refpages/gl4/index.php) to retrieve texels (a.k.a. pixel colors) from our texture

![Texture rendering]({{ site.baseurl }}/assets/images/shaders-texture-workflow.svg)

* To render our texture we need to add another resource operator `TextureResources` in the initialisation branch. Download a texture to use from the [course GitHub](https://github.com/neurogears/st-andrews-2024/tree/main/assets/textures). Double-click `TextureResources` and add a new `ImageTexture` and give it a Name (e.g. `vertGrat`). For the FileName property, use the dialog to select the downloaded texture.
* Subscribe to `DrawCall` and `Scale` the final `DrawMesh` as before.
* Add a `BindTexture` operator after `UpdateUniform` and bind your named texture (set in the `TextureName` field) to the `textureShader` shader.
* You should now see the texture rendered in the display window when the workflow is run.

### **Exercise 8:** Camera textures

We can also dynamically update the texture that is passed to the shader. We can use this to e.g. manipulate a camera feed.

![Dynamic textures]({{ site.baseurl }}/assets/images/shaders-dynamic-texture.svg)

* Add a new `Texture2D` texture to the `TextureResources` operator and name it something like `DynamicTexture`.
* Change the `BindTexture` operator to bind to this new texture instead of the image texture.
* Use a `CameraCapture` and `Flip` operator to produce a camera feed.
* Connect this to an `UpdateTexture` with `TextureName` set to `DynamicTexture` to update the texture on each new camera image.
* You should now see the camera feed rendered in the display window.

**Optional:** Now that you have a camera image rendered via a texture and shader, you can take advantage of GPU acceleration to produce interesting image effects. Try and write a new fragment shader to manipulate the camera image, for example combining the image with a 2nd texture or applying a [sobel filter](https://www.shadertoy.com/view/Xdf3Rf) for edge detection.
  {: .notice--info}
