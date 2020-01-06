# Geometry and Transformation

- [Geometry and Transformation](#geometry-and-transformation)
  - [Introduction Geometry and Transformation](#introduction-geometry-and-transformation)
    - [Scalars](#scalars)
    - [Vectors](#vectors)
      - [Operations](#operations)
      - [Lacking position](#lacking-position)
    - [Points](#points)
      - [Coordinate Systems](#coordinate-systems)
      - [Orthonormal coordinate systems](#orthonormal-coordinate-systems)
      - [Dot product](#dot-product)
      - [Cross product](#cross-product)
    - [Matrix](#matrix)
  - [Linear transformation](#linear-transformation)
  - [Affine transformation](#affine-transformation)

## Introduction Geometry and Transformation

### Scalars

We know what's a scalars

### Vectors

Vectors is a quantity with two measures: direction and magnitude

#### Operations

* Every vector has an inverse -> Same magnitude but point in opposite direction
* Every vector can be multiplied by a scalar
* There is a zero vector
* The sum of 2 vectors is a vector
* The magnitude of a vector is a scalar
* A unit vector is a vector with magnitude 1

#### Lacking position

Vector don't have positions. They are insufficient for geometry. We need points.

### Points

Location in spaces

Operations allowed between points and vector. 

* Points - Points = Vector == Vector + points

#### Coordinate Systems

Every vector can be multiplied by a scalar to scale the vector's magnitude without changing its direction

In 2D, we can represent a vector as a linear combination (or weigthed sum) of any two non-parallel basis vectors.

In 3D, it requires three non-parallel, non coplanar basis vector.

#### Orthonormal coordinate systems

Basis vector taht are unit vector at right angles to each other are called orthonormal (Whatever we are familiar with)

#### Dot product

We can multiply two vectors by  taking the dot product

||a||*||b||*cos(0)

It takes two vectors as arguments, and it returns a scalar.

Used to find the **projection** of one vector onto another

#### Cross product

* Another vector multiplication operation (3D Vectors)

* The direction of a x b is orthogonal to both a and b

### Matrix

* Matrix
* Matrix-matrix multiplication
* Matrix addition
* Matrix muplication by scalar
* Matrix vector multiplication
* Idendity matrix
  
In OpenGL, Vectors are column vector "column major" ordering

## Linear transformation

* Scaling
* Shearing
* Rotation
* Reflection of vectors
* Combinations thereof

All implemented using matrix multiplication

## Affine transformation

All of the linear transformation + translation

It preserves straight lines, parralel lines.

Implementation requires 4x4 matrixes
 
 ## Homogenous Coordinate

 A general object transformation concists of some matrix M and displacement.

 Different rule for points vs vector -> hard to compose
 
 We then add a 4th coordinate to vectors and points
 
 Usually for 3D points we choose w = 1 and for 3D vector = 0

 Benefits? Vectors and points both represented by 4 coord.