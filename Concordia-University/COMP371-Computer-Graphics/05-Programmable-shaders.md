# Programmable shaders

## Programmable shaders

### Programmable hardware

Most modern gpu allow parts of the GPU pipeline to be modified by a dev.

    * increased function with hardware support
    * complex graphics algo. with fast implementation.
    * general computing

    Modified part called **graphich shaders**

### Vertex processor

Responsible for the following op:

    * Vertex transformations
    * Normal transformation & normalization
    * Texture coord. generation and transformation
    * Lighting (per vertex)
    * Color and material

A vertex shader replaces the entire functionaly of the fixed vertex processor.

When writing a shader, one must replace the entire functionality.

No information sharing among vertices -> vertex shaders operate in parallel on all vertices

### Fragment processor

Operates on fragments after the interpolation and rasaterization of vertex data. Allows the following programming:

    * Operation on interpolated vertex values (Phong lighting)
    * Texture access and application
    * fog
    * Color sum, etc.

Again no access to neighbouring fragments! All fragment shaders work in parallel.

Does not replace back-end op.

## Shading Languages

Two dominant API: GLSL (OpenGL), HLSL (DirectX). 

A high-level procedural language (similar to C++)

As of OpenGL 2.0, it is a standard, with a simple gl_XYZ API from OpenGL applications. Former implementations of GLSL

