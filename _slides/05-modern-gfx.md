---
layout: slides
title: Introduction to Modern Graphics Programming
permalink: /slides/gfx/
---

<section data-markdown data-separator="^\n---\n$" data-separator-vertical="^\n--\n$">
<script type="text/template">

![Bonsai](../../assets/images/bonsai-lettering.svg)

### Introduction to modern graphics programming
[neurogears.org/st-andrews-2024](https://neurogears.org/st-andrews-2024)
<table style="width: 100%;">
  <tr>
    <th style="vertical-align: middle; width: 50%; height: 100px; padding-left: 100px">
      <img alt="NeuroGEARS" src="../../assets/images/neurogears.svg"/>
    </th>
    <th style="vertical-align: middle; width: 40%; height: 170px; align: right">
      <img alt="St-Andrews" src="../../assets/images/st-andrews.png"/>
    </th>
  </tr>
</table>

---

#### The generalised graphics pipeline

* Input: data (vertices, textures, shaders)

* Output: pixel colors

---

#### A brief history of computer graphics

</script>
</section>

<section data-background="#000000">
    <section data-background-iframe="https://www.youtube.com/embed/6PG2mdU_i8k?controls=0&amp;enablejsapi=1&amp;autoplay=1&amp;loop=1&amp;showinfo=0&amp;rel=0&amp;html5=1">
      <table style="height: 20%; margin-top: 65%; margin-left: -78px;">
        <tr><th>Beam steering - Tennis for Two, 1958</th></tr>
      </table>
    </section>
    <section data-background-iframe="https://www.youtube.com/embed/LhTTpV5qFrs?t=289&amp;controls=0&amp;enablejsapi=1&amp;autoplay=1&amp;loop=1&amp;showinfo=0&amp;rel=0&amp;html5=1">
      <table style="height: 20%; margin-top: 65%; margin-left: -78px;">
        <tr><th>Software rendering - Elite, BBC Micro, 1985</th></tr>
      </table>
    </section>
    <section data-background-iframe="https://www.youtube.com/embed/MnqLJpgq7jc?controls=0&amp;enablejsapi=1&amp;autoplay=1&amp;loop=1&amp;showinfo=0&amp;rel=0&amp;html5=1">
      <table style="height: 20%; margin-top: 65%; margin-left: -78px;">
        <tr><th>Pushing software rendering - DOOM, 1993</th></tr>
      </table>
    </section>
    <section data-background-iframe="https://www.youtube.com/embed/AdTxrggo8e8?controls=0&amp;enablejsapi=1&amp;autoplay=1&amp;loop=1&amp;showinfo=0&amp;rel=0&amp;html5=1">
      <table style="height: 20%; margin-top: 65%; margin-left: -78px;">
        <tr><th>GPU dominance - Raytraced minecraft, 2020</th></tr>
      </table>
    </section>
    <section data-background-iframe="https://www.youtube.com/embed/usBVx4J4CUM?controls=0&amp;enablejsapi=1&amp;autoplay=1&amp;loop=1&amp;showinfo=0&amp;rel=0&amp;html5=1">
      <table style="height: 20%; margin-top: 65%; margin-left: -78px;">
        <tr><th>Back to basics? - A Short Hike, 2019</th></tr>
      </table>
    </section>
</section>

<section data-markdown data-separator="^\n---\n$" data-separator-vertical="^\n--\n$">
<script type="text/template">

#### The modern graphics pipeline

![Graphics pipeline](../../assets/images/gfx-pipeline.drawio.svg)

---

<!-- .element: data-transition="default none" -->
#### What is a shader?

A program that runs directly on the graphics hardware to transform input data to screen pixels.

--

* Vertex: transforms the positions of vertices, e.g. for clip-space transformation

* Geometry: operates on primitives (lines, triangles) and outputs 0 or more primitives e.g. for level-of-detail transformations

* Fragment: operates on rasterized pixels and outputs a color, e.g. for lighting and other color transformations

---

<!-- .element: data-transition="default none" -->
#### Space

![Graphics pipeline](../../assets/images/Dolphin_triangle_mesh.png)

--

* Objects to be rendered are represented as vertices, and then faces in 3-dimensions.
* In the graphics pipeline we need to 'flatten' this data to a 2D position on screen. 

--

#### Space transformation

![Space transformation pipeline](../../assets/images/ShapeSpaceTransformation.png)

--

* Local space: Geometry defined relative to its own origin, e.g. a single cube in 3D modeling software.
* World space: Many local space objects in a scene need to be defined relative to each other using a global / world origin.

--

* Camera space: The vertices that we want to see on screen are determined by our camera / viewport in the 3D world. We therefore need to know where these vertices are relative to our camera position and orientation.

--

* Clip space: Aside from position and direction, the viewing properties (e.g. field-of-view) of the camera affect which vertices we want to see on screen.
* Unlike a 'real' camera, we can also decide to exclude vertices that are too close / far from the viewing plane.
* The projection transform places our vertices into clip-space, so-called because we can decide at this stage which polygons will be on screen, and whether we need to 'clip' anything outside the screen.

--

* Perspective divide: At this stage we may also want to apply perspective mapping, e.g. to make objects further away appear smaller.

--

* Normalised device coordinates: Once these steps have been performed our vertices are in a format which can be rasterized by the graphics hardware.
* Screen position: Finally we need to transform into a 2D screen space, according to the size and resolution of the viewport window display.

---

#### OpenGL

<!-- .element: data-transition="default none" -->
* What is OpenGL?

* An API for communicating with hardware renderers (GPUs)

    * Low-level definition of the graphics pipeline

    * Allocation of data and resources to the GPU like shaders, materials etc.

* Powerful, flexible but...

--

<!-- .element: data-transition="default none" -->
#### OpenGL
```
// CPP program to render a triangle using Shaders
#include <GL\freeglut.h>
#include <GL\glew.h>
#include <iostream>
#include <string>
 
std::string vertexShader = "#version 430\n"
                           "in vec3 pos;"
                           "void main() {"
                           "gl_Position = vec4(pos, 1);"
                           "}";
 
std::string fragmentShader = "#version 430\n"
                             "void main() {"
                             "gl_FragColor = vec4(1, 0, 0, 1);"
                             "}";
 
// Compile and create shader object and returns its id
GLuint compileShaders(std::string shader, GLenum type)
{
 
    const char* shaderCode = shader.c_str();
    GLuint shaderId = glCreateShader(type);
 
    if (shaderId == 0) { // Error: Cannot create shader object
        std::cout << "Error creating shaders";
        return 0;
    }
 
    // Attach source code to this object
    glShaderSource(shaderId, 1, &shaderCode, NULL);
    glCompileShader(shaderId); // compile the shader object
 
    GLint compileStatus;
 
    // check for compilation status
    glGetShaderiv(shaderId, GL_COMPILE_STATUS, &compileStatus);
 
    if (!compileStatus) { // If compilation was not successful
        int length;
        glGetShaderiv(shaderId, GL_INFO_LOG_LENGTH, &length);
        char* cMessage = new char[length];
 
        // Get additional information
        glGetShaderInfoLog(shaderId, length, &length, cMessage);
        std::cout << "Cannot Compile Shader: " << cMessage;
        delete[] cMessage;
        glDeleteShader(shaderId);
        return 0;
    }
 
    return shaderId;
}
 
// Creates a program containing vertex and fragment shader
// links it and returns its ID
GLuint linkProgram(GLuint vertexShaderId, GLuint fragmentShaderId)
{
    GLuint programId = glCreateProgram(); // create a program
 
    if (programId == 0) {
        std::cout << "Error Creating Shader Program";
        return 0;
    }
 
    // Attach both the shaders to it
    glAttachShader(programId, vertexShaderId);
    glAttachShader(programId, fragmentShaderId);
 
    // Create executable of this program
    glLinkProgram(programId);
 
    GLint linkStatus;
 
    // Get the link status for this program
    glGetProgramiv(programId, GL_LINK_STATUS, &linkStatus);
 
    if (!linkStatus) { // If the linking failed
        std::cout << "Error Linking program";
        glDetachShader(programId, vertexShaderId);
        glDetachShader(programId, fragmentShaderId);
        glDeleteProgram(programId);
 
        return 0;
    }
 
    return programId;
}
 
// Load data in VBO and return the vbo's id
GLuint loadDataInBuffers()
{
    GLfloat vertices[] = { // vertex coordinates
                           -0.7, -0.7, 0,
                           0.7, -0.7, 0,
                           0, 0.7, 0
    };
 
    GLuint vboId;
 
    // allocate buffer space and pass data to it
    glGenBuffers(1, &vboId);
    glBindBuffer(GL_ARRAY_BUFFER, vboId);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
 
    // unbind the active buffer
    glBindBuffer(GL_ARRAY_BUFFER, 0);
 
    return vboId;
}
 
// Initialize and put everything together
void init()
{
    // clear the framebuffer each frame with black color
    glClearColor(0, 0, 0, 0);
 
    GLuint vboId = loadDataInBuffers();
 
    GLuint vShaderId = compileShaders(vertexShader, GL_VERTEX_SHADER);
    GLuint fShaderId = compileShaders(fragmentShader, GL_FRAGMENT_SHADER);
 
    GLuint programId = linkProgram(vShaderId, fShaderId);
 
    // Get the 'pos' variable location inside this program
    GLuint posAttributePosition = glGetAttribLocation(programId, "pos");
 
    GLuint vaoId;
    glGenVertexArrays(1, &vaoId); // Generate VAO
 
    // Bind it so that rest of vao operations affect this vao
    glBindVertexArray(vaoId);
 
    // buffer from which 'pos' will receive its data and the format of that data
    glBindBuffer(GL_ARRAY_BUFFER, vboId);
    glVertexAttribPointer(posAttributePosition, 3, GL_FLOAT, false, 0, 0);
 
    // Enable this attribute array linked to 'pos'
    glEnableVertexAttribArray(posAttributePosition);
 
    // Use this program for rendering.
    glUseProgram(programId);
}
 
// Function that does the drawing
// glut calls this function whenever it needs to redraw
void display()
{
    // clear the color buffer before each drawing
    glClear(GL_COLOR_BUFFER_BIT);
 
    // draw triangles starting from index 0 and
    // using 3 indices
    glDrawArrays(GL_TRIANGLES, 0, 3);
 
    // swap the buffers and hence show the buffers 
    // content to the screen
    glutSwapBuffers();
}
 
// main function
// sets up window to which we'll draw
int main(int argc, char** argv)
{
    glutInit(&argc, argv);
    glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE);
    glutInitWindowSize(500, 500);
    glutInitWindowPosition(100, 50);
    glutCreateWindow("Triangle Using OpenGL");
    glewInit();
    init();
    glutDisplayFunc(display);
    glutMainLoop();
    return 0;
}
```

--

<!-- .element: data-transition="default none" -->
#### OpenGL
![Triangle](../../assets/images/triangle.png)

--

---

#### Bonsai.Shaders

* Abstract away some low-level OpenGL boilerplate - window creating, texture binding etc.
* Allows manipulation of the graphics pipeline directly in Bonsai.

---

</script>
</section>
