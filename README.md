# Mesh Deformation toolkit
Full mesh deformation toolkit inside UE4, including tools (Move, Scale, Rotate, Spherize etc) based on popular 3d modelling packages, and a set of utilities to control which parts of a mesh will be affected.

In order to allow complex effects this supports:
* Multiple transformations, either using the same selection set or different ones.
* Combining selections for greater control.
* Can be extended in Blueprint to add custom transformations and selection controls.

**At present this should be considered alpha quality as code needs more bullet-proofing, documentation, and any and all APIs may change**

# Introduction
Any 3d artist has done mesh deformation, even if they're not familiar with the term.

It's where you have some existing geometry, select some or all of the vertices, and move them about using the tools provided.

Mesh deformation isn't create or destroy geometry, and it's not changing the triangle data connecting the vertices so tearing and so on isn't supported, but it is a flexible toolset for manipulating geometry.

This plugin, the Mesh Deformation Toolkit (MDT for short) allows you to do this type of geometry manipulation inside of Unreal Engine via Blueprint.

This is done by providing a UE4 Component called **MeshDeformationComponent** which allows you to store and deform mesh  geometry, using the selection and transformation nodes provided, and then output it to your UE4 level.  All inside Unreal Engine, and available either inside the editor or at runtime.

If you like SplineMeshComponent you should love MeshDeformationComponent.

## Basic Workflow
The workflow for this plugin is based on that a 3d artist would use in their tools.

### Step 1: Load the geometry
In a 3d package this is where you either use the tools provided to create a base mesh, or load one in.

In the plugin this is done by the "Load" nodes, although at present there is only one of these- "Load from Static Mesh" which uses a standard StaticMesh.

### Step 2: Select the vertices to modify
In your 3d package this is where you drag your mouse over the vertices and choose those you want to move.

In the plugin this is the various "Select" nodes which each allow a different type of selection- such as vertices near a provided position, or near a provided spline path.  Similar to many 3d packages this selection isn't an all-or-nothing selection, some points can be 'partly selected' which makes it easier to gets smoother results.

There are also selection modifiers which allow you to invert selections, apply easing functions to make selections smoother, or combine selections by adding, subtracing, blending, or more.

### Step 3: Transform the vertices
There are a lot of tools for this in a 3d package, some allowing basic operations such as Move, Scale, Rotate, and other stranger ones such as Inflate which moves points along their normals, or Spherize which blends a mesh towards a spherical shape.

This plugin provides a set of transform nodes based on the tools 3d packages provide, each of which can also take a selection to control their influence on the vertices.

A selection can also be used to drive multiple transformations so if you want to move your selected points upwards and then rotate them that's fine.

### Step 4: Rinse and repeat...
It's rare that you'll get exactly the shape you want with a single transformation, you'll want to to keep selecting and moving different parts of the mesh.

This can be done by just chaining selections/transformations together.

#### Step 5: Save the result
In a 3d package this is where you save your results to disk ready for rendering, or to import into another tool such as UE4.

In the plugin this is the "Save" nodes.  Again at present there's only one of these, "Update Procedural Mesh Component" (*NB:* This is going to get renamed to Save To Procedrual Mesh Component) which takes the transformed geometry and writes it out to one of Unreal's ProceduralMeshComponents which can then be added to the scene.

# Installing the plugin
When the plugin hits public beta I'll be releasing compiled plugins for Windows and Mac, and possibly other platforms too.

For now though to get it working you'll need the *Plugins/MeshDeformationToolkit* directory from this repo and copy it to the same path in your own project, then when you run UE4 you should be able to enable/compile the plugin (This will need a C++ compiler).

# Node Dictionary
Here is a list of all of the nodes the system provides, broken down into six categories, five of which roughly correspond to the steps listed in the intro and 'Utility' which is a grab-bag of everything else:

1. *Loading Geometry Data*.  Before we do anything else we need some geometry data to operate on.
2. *Select Vertices*.  While we can do some useful operations on every vertex that makes up the mesh the ability to only change parts of the mesh is much more powerful, and for that we need to select the vertices to control the operation.  These selections aren't limited to a simple yes/no, it's possible to have vertices which are partially selected to allow an operation to be applied at varying strengths on different parts of the mesh.
3. *Modifying And Combining Selections*.  For even greater control over which parts of the mesh we're affecting it's possible to take selections and alter them to get the exact control needed.
4. *Transforming Vertices*.  The actual deformations, whether we're moving and rotating points or something stranger this is where the actual work of changing the geometry happens.
5. *Output Data*.  There's no point in changing the geometry if we then don't do something with it and these nodes are where that happens.
6. *Utility*.  This is everything else, including tools which allow you to build your own selection and transform tools.

## Load Geometry Data
All of these nodes are provided on the Mesh Deformation Component, and also their stored geometry.

* *Load From Static Mesh*: Use a StaticMesh as the source of the original geometry. (Demo: *PassthroughDemo/PassthroughMap*, and used in many others)

## Select Vertices
Each of these functions returns a Selection which can then be either be either modified further, or passed into any of the nodes for transforming vertices to control their behavior.

* *Select All*: Select all vertices at full strength
* *Select By Noise*: Select vertices based on a noise function, useful for terrain or adding controlled randomness to a model. *This can return values outside of the standard zero to one range, if this causes problems use a 'Remap By Range' node to remap between zero and one.* (Demo: *SelectionDemo/SelectByNoiseTranslateMap*, *SelectionDemo/SelectByNoiseVoronoiTranslateMap*)
* *Select By Normal*: Selection based on a vertex's normal direction. (Demo: *SelectionDemo/SelectByNormalTranslateMap*)
* *Select By Section*: Selection based on the sections which make up a mesh. (Demo: *SelectionDemo/SelectBySectionTranslateMap*)
* *Select By Texture*: Use a channel from an RGB texture to select vertices.
* *Select In Volume*: Simple selection based on whether a vertex is within an axis-aligned volume. (Demo: *SelectionDemo/SelectByVolumeTranslateMap*)
* *Select Linear*: Selection based on a gradient between two provided points.
* *Select Near*: Select all vertices near a provided point. (Demo: *SelectionDemo/SelectNearTranslateMaterialBallMap*, *SelectionDemo/SelectNearTranslatePlaneMap*)
* *Select Near Line*: Select all vertices near a provided line, which can be an infinite line. (Demo: *SelectionDemo/SelectNearLineTranslateMap*, *SelectionDemo/SelectNearInfiniteLineTranslateMap*)
* *Select Near Spline*: Select all vertices near a [Spline Component](https://docs.unrealengine.com/latest/INT/Engine/BlueprintSplines/Overview/). (Demo: *SelectionDemo/SelectNearSplineMap*)

## Modifying And Combining Selections
These aren't actually on the Mesh Deformation Component and instead are provided in a Blueprint Function Library.

All of these return a new Selection and don't modify the ones provided to them.  The short descriptions below are often phrased as though they do but this is because writing 'Return a Selection which is the provided Selection' in each one would make them a lot harder to read.

* *Clamp (Selection, Float, Float)*: Clamps the range of a Selection between the minimum and maximum provided.
* *Ease*: Remaps a selection by applying a configurable easing function to each value.  Useful for taking the linear fall-off from a selection and turning it into something smoother and more natural looking.
* *Float - Selection*: Subtract all value in a selection from a constant value, providing a simple reverse and remap.
* *Float / Selection*: Divide a constant value by all values in a selection.
* *Lerp (Selection, Float, Float)*: Blends a Selection between its original value and a constant value with a set strength.
* *Lerp (Selection, Float, Selection)*: Blends a Selection between its original value and a constant value with a strength controlled by a second Selection.
* *Lerp (Selection, Selection, Float)*: Blend two Selections together with a given strength.
* *Lerp (Selection, Selection, Selection)*: Blend two Selections together with a strength controlled by a third Selection.
* *Max (Selection, Float)*: Make sure all values in a Selection are at least equal to a given value.
* *Max (Selection, Selection)*: Combine two Selections together by taking the maximum value from each one of them.
* *Min (Selection, Float)*: Make sure all values in a Selection are at most equal to a given value.
* *Min (Selection, Selection)*: Combined two Selections together by taking the mium value from each of them.
* *Randomize (Selection, Float, Float)*: Randomize the values in a Selection between the minimum and maximum values provided.
* *Remap To Curve (Selection, Curve)*:
* *Remap To Range (Selection, Float, Float)*: Remaps a selection between the minimum and maximum values provided.
* *SelectionSet + Float*: Adds the given value to all values in a Selection.
* *Selection + Selection*: Combine two Selections together by adding their values.
* *Selection - Float*: Subtract a given value from all values in a Selection.
* *Selection - Selection*: Combine two Selections by subtracting all values in the second Selection from the values in the first Selection.
* *Selection * Float*: Multiply all values in a Selection by a given value.
* *Selection * Selection*: Combine two Selections by multiplying their values.
* *Selection / Float*: Divide all of the values in a Selection by the given value.
* *Selection / Selection*: Combine two Selections by dividing all the values from the first Selection by those of the second Selection.
* *Set (Selection, Float)*: Sets all values in a Selection to the given value.

## Transforming Vertices
All of these transform operations can be controlled by providing an optional Selection.  While the actual use of the Selection can vary method nodes it's intended that each one uses it in the most obvious and flexible way for that node's own purpose.

* *Conform*: Project all of the vertices in a provided direction as though it were being draped over other geometry in the level.  This is a difficult node to get to grips with but is very useful for making roads which follow the underlying terrain and similar effects.
* *Fit To Spline*: Bend the geometry to follow a provided [Spline Component](https://docs.unrealengine.com/latest/INT/Engine/BlueprintSplines/Overview/), with controls for the profile of the geometry for more useful effects.  This is more powerful than UE4's own [Spline Mesh Component](https://docs.unrealengine.com/latest/INT/Engine/BlueprintSplines/Overview/) in that it follows an entire curve rather than just having the two control points at the ends.
* *Inflate*: Move all of the vertices in or out along their own normals.
* *Jitter*: Applies random position jitter to the vertices.  This is a fairly crude effect and generally you're better off using Select By Noise and Translate for greater control.
* *Lerp (MeshDeformationComponent, Float)*: Blend this with the geometry stored in another MeshDeformationComponent with a given strength.
* *Lerp (MeshGeometry, Float)*: Blend this mesh with another with a given strength.  *NOTE: This should be exposed on MeshDeformationComponent too*.
* *Lerp (Vector)*: Blend all vertices towards a single given point.
* *Rotate*: Rotate vertices using a standard UE4 [Rotator](https://docs.unrealengine.com/latest/INT/BlueprintAPI/Math/Rotator/index.html).
* *Rotate Around Axis*: Rotate all of the points around a given axis and center of rotation.  This is more difficult to use than Rotate, but is also more flexible.
* *Scale*: Scale vertices by the vector provided.
* *Scale Along Axis*: Scale vertices along a provided axis and center of scale
* *Spherize*: Blend all vertices towards a position on a sphere with the provided center and radius.
* *Transform*: Apply the provided transform matrix to all vertices.
* *Translate*: Add a provided vector to all vertices.  This is the basic Move operation.

## Output Data
* *Save To Procedural Mesh Component*: Write the geometry data to a standard UE4 [Procedural Mesh Component](https://wiki.unrealengine.com/Procedural_Mesh_Component_in_C%2B%2B:Getting_Started).

## Utility

* *Get Bounding Box*: Return the bounds of the mesh as a [Box](https://docs.unrealengine.com/latest/INT/API/Runtime/Core/Math/FBox/index.html) which.
* *Get Summary*: A simple text string summary of how many sections, vertices and triangles are in the current geometry.
* *Has Geometry*: Returns whether we have any geometry loaded or not
* *Get Total Triangle Count*: Return the total number of triangles in the mesh.
* *Get Total Vertex Count*: Return the total number of vertices in the mesh.

# How It Works
When everything is working and finished I will be taking the time to document how exactly this plugin works, including how you can extend it yourself by writing new selection/transform nodes either in C++ or Blueprint.

For now though I can only point you towards [a tutorial I wrote which started all of this](http://normalvector.com/tutorials/low_poly_world/) which has the basic approach to deforming geometry (In pure Blueprint too!), and add the following quick notes.
* Geometry is basically stored in the format which ProceduralMeshComponent wants, and so a mesh contains a list of sections- each of which has it's own vertex list.  It is this vertex list which the plugin manipulates.
* All any Selection node does is iterate over all of the vertices in a mesh and apply a selection rule to each to obtain the selection weight, storing these in an array in SelectionSet.  The array is ordered the same as the mesh vertices to prevent any other metadata being needed.
* The selection modification nodes just take one (or more) SelectionSets and return another SelectionSet with the weights tweaked as needed.
* Every Transform node is actually currently implemented as a method on the mesh geometry which iterates over the vertices in the mesh and applies the transformation to each of them in order.  Selections are handled by basically acting as a lerp between the original position and the fully transformed position, although the nature of the lerp can vary between methods (Translate has a straightfoward spatial lerp, but Rotate actually lerps the rotational angle for better results).  Most of these are implemented in a Blueprint Library class as they're utility functions rather than being called on a class.
* The MeshDeformationComponent itself just contains a set of mesh geometry and offers BP-callable selection and transform nodes which just delegate the call to the mesh.

# Code documentation
All C++ code has been documented using [Doxygen](http://www.stack.nl/~dimitri/doxygen/) together with [my Doxygen source filter to remove UE4 macros](https://github.com/normalvector/ue4_doxygen_source_filter), and will be made available as API docs online at some point.

This is a [test link to internal docs making sure GitHub works as expected](./docs/test_doc.md).
TODO: REMOVE THIS

# Thanks Go To..
[Jordan Peck](https://github.com/Auburns) for his splendid [FastNoise](https://github.com/Auburns/FastNoise) C++ library, which is used for all of the noise generation in the plugin.

# ToDo List
The ToDo list for this project is on [Trello](https://trello.com/b/eWCcepmW/mesh-deformation-component).

This also doubles as a changelog with the 'Done' list.

# LICENSE ([MIT License](https://en.wikipedia.org/wiki/MIT_License))

Copyright 2017 Paul Golds.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
