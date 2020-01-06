# Lightning and Shading

## Global Illumination

Group of algos. for realistic lightning in 3D scenes:

    * ray tracing
    * radiosity
    * photon mapping

Features

    * Follow light rays through a scene
    * Direct + indirect lightning -> Realistic lighting
    * Accurate, but expensive (off-line)

## Local Illumination

Approximate model

Local interaction between light, surface, viewer.

Phong model: fast supported in "fixed pipeline" OpenGL

Vertex|Fragment shaders