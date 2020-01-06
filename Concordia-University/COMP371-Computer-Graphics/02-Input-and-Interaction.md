# Input and interaction

- [Input and interaction](#input-and-interaction)
  - [I and I](#i-and-i)
    - [Client - Server model](#client---server-model)
    - [Input modes](#input-modes)
      - [Request mode](#request-mode)
      - [Sample mode](#sample-mode)
      - [Event mode](#event-mode)
        - [Event types](#event-types)
    - [Callbacks + GLFW](#callbacks--glfw)
      - [GLFW error handling](#glfw-error-handling)
      - [GLFW GLFW main event loop](#glfw-glfw-main-event-loop)
  - [Double buffering](#double-buffering)
  - [Controlling Speed](#controlling-speed)
  - [Hidden surface removal](#hidden-surface-removal)
    - [Object space algo](#object-space-algo)
      - [Painter's algorithm](#painters-algorithm)
        - [Easy cases](#easy-cases)
        - [Hard cases](#hard-cases)
        - [Pros](#pros)
        - [Cons](#cons)
    - [Image-space algo](#image-space-algo)
      - [z-buffer algorithm](#z-buffer-algorithm)
        - [Pros](#pros-1)
        - [Cons](#cons-1)
        - [OpenGL + GLFW](#opengl--glfw)

## I and I

### Client - Server model

This model was popularized by the X Window System

* Servers perform task for clints
* Workstation with a display is a graphich server

### Input modes

Input devices contain a **trigger** which can be used to send a signal to the OS (Button on mouse, Pressing a key)

When triggered, input devices return information (pos. info for mouse, keyboard returns ASCII)

The application can obtain the measure from three mode: 

* Request
* Sample
* Event

#### Request mode

Input provided to program only when user triggers the device (keyboard)

* a C program requires a char input -> `scanf("%d", &x)`

#### Sample mode

* Input is immediate
* No trigger is needed
* The user must have positioned the pointing device/entered data using the keyboard before the function call

Appropriate for character controls where smooth movement is required.

#### Event mode

The mode we will be using in this class

* Most systems have more than one input device, each of which can be triggered at an arbitrary time by a user
* Each trigger generates an event whose measure is put in an event queue which can be examined by the user program

##### Event types

* Window: resize, expose, iconify
* Mouse: click/wheels
* Motion: move mouse, joystick
* Keyboard: press/release keys

### Callbacks + GLFW

* Programming interface for event-driven input
* Define callback function for each type of event the graphics system recognizes
* The user-supplied function is executed when the event occurs

```C++
//The GLFW function for registering your callback function for mouse movement is
glfwSetCursorPosCallback(window, myCursorMovedFunction);

//User implementation of the callbackk function with the same name/input params
static void myCursorMovedFunction(GLFWwindow* window, double xpos, double ypos){ }
```


Callback input typedefs are found @ http://www.glfw.org/docs/lastest/group__input.html

#### GLFW error handling

```C++
//Define a callback function
void error_callback(int error, const char* description){
    fputs(description, stderr)
};

//Register the error callback
glfwSetErrorCallback(error_callback)
```

#### GLFW GLFW main event loop

* The main loop processes the events in the event queu
* Processing events will cause the window and input callbacks associated with those events to be called. 

`glpwPollEvents()` processes only those events that are already in


## Double buffering

When [re]drawing stuff on the display, we must do so at a high enough rate that we cannot notice the clearing and redrawing

* screen refreshing: common 60-100 Hz (times/s)
* if frame buffer contents are unchanged -> OK
* if frame buffer contents change -> flickering, undesirable artifacts/distortions

Solution -> Use 2 different frame buffers

* Front: always the one displayed
* Back: one into which we draw

The buffers are swapped only after an explicit function call by the aplication program. 
`glfwSwapBuffers(GLFWwindow * window);`

## Controlling Speed

GPU can render millions of primitives per second. Running a simple program with a rotating cube will result in the cube being rendered thousands of times per second -> Blur on the display

Solution: Use a timer

* use timing mechanisms provided by libs or OS to put delays
* lock the swapping of buffers to the screen refresh rate `glfwSwapInterval(1)` blocks swapping until he monitor has done at least one vertical redraw since the last buffer swap. I.E. puts an upper limit on the frame rate at the monitor's refresh rate.

## Hidden surface removal 

Objective: remove surfaces which are not visible to the viewer. Many solutions have been proposed. These can be categorized into: 

* Object-space algorithms
* Image-space algorithms

### Object space algo

Use pairwise testing between polygons (objects)

Worst-case complexity O(n^2) for n polygons

#### Painter's algorithm

Render polygons in a back to front order so that polygons behind others are simply painted over. 

This requires a depth sort first. 

* O(nlogn) calculation for ordering
* Not every polygon is either in front or behind all other polygons -> order polygons and deal with easy cases first, harder later

##### Easy cases

1) Polygon lies behind all other polygon -> can render
2) Polygons overlap in z but not in x,y -> can render independantly

##### Hard cases

1) Overlap
2) Cycle overlap
3) Penetration

When this happens, you need to break the polygons and redo the process on them.


##### Pros

* Simple
* Can handle transparency well
* In special case you don't have to sort

##### Cons

* Cannot handle complex geometry well
* Not supported by OpenGL, need to define the algo 
  
### Image-space algo

* Look at each projector (nm for an n*m buffer) and find closest of k polygons
* complexity O(nmk)
* Ray tracing
* z-buffer

#### z-buffer algorithm

Use a buffer called the z-buffer (or depth buffer) to store the depth of the closest object at each pixel found so far, as we render each poly, compare the depth of each pixel to the depth in z-buffer. If less, place shade of pixel in color buffer and update z-buffer.

##### Pros

* Simple
* Independent of geometric primitives

##### Cons

* Memory intensity
* Tricky to handle transparency / blending
* Depth-ordering artifacts

OpenGL implements z-buffering. It must be enabled.

##### OpenGL + GLFW

Ask a depth buffer when you create a window `glfwWindowHint(GL_DEPTH_BITS,24)`

Call to `glEnable(GL_DEPTH_TEST)` in your progrma's init routine, after a context is created and made current

Set the depth function using `glDepthFunc(GL_LESS)`

Every time you draw call `glClear(GL_DEPTH_BUFFER_BIT);`
