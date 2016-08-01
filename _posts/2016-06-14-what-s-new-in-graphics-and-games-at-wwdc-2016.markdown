---
published: true
title: What's new in graphics and games at WWDC 2016
layout: post
---
Like every year, `June` is my favorite month of the year for several reasons, but __WWDC__ is most likely the top one! Watching the opening `Keynote` and the `Platforms State Of The Union` sessions yesterday revealed a plethora of new features and even a few new frameworks. In this article, I am only going to focus on what's new in the __Graphics and Games__ track.

Let's start with __Metal__, obviously. By far, the hottest and most anticipated feature is support for __Tessellation__ which enables 3D apps and games to render more details by efficiently describing complex geometry to the `GPU`. Another feature is __Function Specialization__ which helps with creating a collection of functions particularly optimized to handle material and light combinations in a scene. Also new this year are the __Resource Heaps and Memoryless Render Targets__ for finer-grained control of resource allocation and performance optimization in `iOS` and `tvOS`. Finally, the __Metal System Trace__ is a `macOS`-only feature that helps us analyzing the graphics pipeline by profiling the interaction between the `CPU` and the `GPU`, thus helping us finding performance optimization points for Metal-based apps.

The __Model I/O__ framework brings support for the __USD__ file format. The __MDLMaterialPropertyGraph__ class now makes it easier to support runtime procedural changes to models. Also, the __MDLVoxelArray__ class now adds support for signed distance fields. Finally, you can now add assisted light probe placement through the __MDLLightProbeIrradianceDataSource__ protocol.

The __GameplayKit__ framework brings us __Procedural noise generation__ that can be used to generate richer game worlds, more sophisticated textures, more realistic to camera movements. Next,
__Spatial partitioning__ lets us partition game world data so that it can be searched efficiently. Also, the new __Monte Carlo strategist__ helps us model games where exhaustive computation of possible moves is difficult. The new __decision tree API__ can enhance our game-building AI when adopting decision-tree learning to generalize behavior based on data mining of logged player actions. 

The __ReplayKit__ framework introduces support for __tvOS__ and for __broadcasting__ so we can broadcast recorded media through a third-party site. 

The __SceneKit__ framework introduces a new __Physically Based Rendering__ system that empowers us to create more realistic results with simpler asset authoring. Also, the new HDR features and effects help us creating even more realism.

The __SpriteKit__ framework introduces a new tilemap solution to support square, hexagonal, and isometric tilemaps. The Xcode editor also provides support for organizing the tiles and the tilemap.

The __Accelerate__ framework introduces support for `quadrature` (integral calculus), basic functions for constructing neural networks, and geometric predicate functions to test for object intersections.

The __Core Image__ framework now allows us to insert custom processing into a `Core Image` filter graph. `Core Image` kernel code can now request a specific output pixel format. Finally, `Core Image` adds five new filters to the existing filter collection.

Stay tuned for more news, and have a great `WWDC`!