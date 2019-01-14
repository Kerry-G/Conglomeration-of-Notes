# Introduction of Computer Graphics

- [Introduction of Computer Graphics](#introduction-of-computer-graphics)
  - [OpenGL API](#opengl-api)
    - [What is OpenGL?](#what-is-opengl)
    - [State machine](#state-machine)
    - [Related library](#related-library)
    - [Recent changes in OpenGL paradigm](#recent-changes-in-opengl-paradigm)
  - [Grapichs Pipeline - simplified version](#grapichs-pipeline---simplified-version)
    - [Pipelining](#pipelining)
    - [Step 1 : Vertex Processor](#step-1--vertex-processor)
      - [Transformation](#transformation)
      - [Vertex color](#vertex-color)
    - [Step 2: Clipping and primitive assembly](#step-2-clipping-and-primitive-assembly)
    - [Step 3: Rasterization](#step-3-rasterization)
    - [Step 4: Fragment Processor](#step-4-fragment-processor)
  - [Graphics pipeline - extended version](#graphics-pipeline---extended-version)
  - [Primitives & Attributes](#primitives--attributes)
    - [OpenGL Primitives types](#opengl-primitives-types)
      - [Winding Order](#winding-order)
      - [Polygons issues](#polygons-issues)
    - [OpenGL Attributes](#opengl-attributes)
      - [Colors](#colors)
        - [RGB - Red, Green Blue,](#rgb---red-green-blue)
        - [HSV - Hue, Saturation, Value](#hsv---hue-saturation-value)

## OpenGL API

### What is OpenGL?

* Low-level graphics lib
* Originated from SGI's GL in '92
* Current version 4.6 (as in May 14 2018)
* Managed by Khronos Group (non-profit consortium)
* API is governed by Architecture Review Board (part of Khronos)
* Used in broadcasting, CAD/CAM/CAE, entertainment, medical imaging, VR.
* cross-platform

### State machine

Give it orders to set the current state of any one of its internal variables, or to query for its current status

State variables: color, camera position, light position, material properties, etc.

The current state persists until set to new values by the program.

### Related library

* OpenGL (GL) - Core graphics - functions begin with gl

* OpenGL Framework Library (GLFW) - provides programmers with the ability to create and manage windows and OpenGL contexts, as well as handling of input devices and event - functions begin with glfw

* OpenGL Extension Wrapper Library (GLEW) - provides efficient run-time mechanisms for determining which OpenGL extensions are supported on the target platform - function begin with glew

* GLM - OpenGL Mathematics - C++ math lib based on the OpenGL Shading Language (GLSL)

* OGRE - Object-Oriented Graphics Rendering Engine - scene-oriented, real-time, 3D rendering engine in C++. 

### Recent changes in OpenGL paradigm

Version 3.1 and less; OpenGL was using immediate mode

* Primitives were not stored in the system but rather passed through the system for possible rendering as soon as they become available
* Each time a vertex is specified in app, it is sent to the GPU
* Creates data transfer bottleneck between CPU/GPU
* The scene is re-drawn for each frame

Version 3.1 and more; OpenGL was using retained mode

* Goal: Data transfers from CPU to GPU are minimized
* Solution: put all vertex and attribute data in an array, send the array to the GPU to be rendered each time
* Revised solution: put all vertex and attribtue data in an array, send the array to the GPU for multiple renderings.

## Grapichs Pipeline - simplified version

All vertices must be processed efficiently in a similar manner to form an image in the framebuffer

Example: a scene contains a set of objects -> Each object comprises a set of geometric primitives -> Each primitive comprises a set of vertices -> A complex scene can hae millions of vertices that define the objects


### Pipelining

* Pipielining is similar to an assembly line in a car plant
* Efficiently implementable in hardware
* Each stage can employ multiple specialized processors, working in parallel, buses between stages
* nb processors per stage, bus bandwidths are fully tuned typical graphics use
* latency vs throughput
  
### Step 1 : Vertex Processor

The GPU is optimized to render triangles

* Every objects can be drawn out of triangles
* Curved surface can be approx. using triangles
* Each triangles is 3 vertices
* Each vertex we define a position and other attributes

The vertex processor have 2 major functions: 

* 1- Transformation
* 2- Vertex color

#### Transformation

Transformations are functiosn that map points from one place to another and are central to understanding 3D computer graphics

Almost every step in the rendering pipeline involves a change of coord. systems. The best way to implement transformations is with matrix operation (in 3D, 4x4 matrixes)

The position of each vertex will go through a sequence of transformations from the Model Coordinate System to the Screen Coordinate System. Every vertex on a model is transformed using a transformation matrix taking into account the position of the model in the world, the camera position and projection parameters.

This matrix is called the **Model-View-Projection Matrix**

#### Vertex color

### Step 2: Clipping and primitive assembly

* No imaging system can see the whole world at once -> use a clipping volume to define visible area
* Clipping is done mostly automatically by defining the viewport
* Clipping is performed on primitives
* Primitive assembly must be done before

3 Possibles cases

* Visible -> Contine processing
* Non-visible -> Stop processing
* Partially visible -> continue with visible part

The projection of objects in the clipping volume appear in the final image. Projections are represented as transformations (orthographic vs perspective)

### Step 3: Rasterization 

The primitives that emerge from the clipper are still represented in terms of their vertices and must be further processed to generate pixels in the frame buffer

The rasterization determines which pixels should be colored in the frame buffer

Transforms vertices to window coordinates

* Vertex values are interpolated inside the triangle
* Anti-aliasing

The output is a set of **fragments** for each primitive

* A fragment is a potential pixel which carries information

### Step 4: Fragment Processor

The fragment processor takes in the fragments generated by the rasterizer and updates the pixels in the frame buffer

Processing examples

* Fragments may not be visible
* Color may be altered because of texture or bump mapping
* Translucent effect by combining the fragment color with the pixel color

## Graphics pipeline - extended version

Shaders

* Vertex: transformations, vertex color
* Tessellation: subdivisions - vertex level
* Geometry: takes one primitive and can output 0 or + primitives, layered rendering
* Fragment: effects, fragment color

## Primitives & Attributes

### OpenGL Primitives types

* Polygons (GL_POLYGON): successive vertices define line segments, last vertex connects to first
* Triangles and Quadrilaterals (GL_TRIANGLES, GL_QUADS) succesive verticies of 3 or 4 points for triangles or quads
* Strips and Fans (GL_TRIANGLES_STRIP, GL_TRIANGLES_FAN) are joined triangles/quads
* Lines Strips and Loops (GL_LINE_STRIP, GL_LINE_LOOP) are similar to polygons but not filled

#### Winding Order

Polygons in OpenGL are usually one-sided, meaning they can only be viewed from their "front" side. If you try to draw a polygon with its "back" side, the result is nothing.

To get this behavior Backface Culling must be enabled `glCullFace(GL_BLACK)`;

#### Polygons issues

OpenGL will convet quads and polygons into triangles and only display triangles.

* Simple: edges cannot cross
* Convex: All points on line segment between two points in a polygon are also in the polygon
* Flat: all vertices are in the same plane

Application program must teessellate a concave polyugon into triangles, since 4.1 OpenGL contains a tessellator

Convexity and simplicity is expensive to test -> OpenGL handles only simple and convex polygons

Behavior of OpenGL implementation on disallowed polygons is **undefined**

### OpenGL Attributes 

Attributes are used to define the appearance of objects: Color, Size, Stipple pattern, polygon mode, display as filled, display edges, display vertices

#### Colors

Color is the result of interaction between physical light in the environment and our visual system.

##### RGB - Red, Green Blue,

* Each color component is stored separately
* Usually 8 bits per component in buffer, 32 for HDR images
* Conveniant for display

##### HSV - Hue, Saturation, Value

* Hue: What color
* Saturation: How far away from gray
* Value: How bright

There are many others color mode i.e. CMYK