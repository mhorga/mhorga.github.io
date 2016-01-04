---
published: false
title: Introducing the Metal framework
layout: post
---
As I promised last time, I am starting today a new series about the __Metal framework__ for accessing the `GPU (Graphics Processing Unit)` inside your machine. Here are the main advantages of using `Metal`:

- provides the lowest overhead access to the `GPU`, hence it reduces all bottlenecks caused by data transferring between the `CPU` and `GPU`. 
- provides up to __10__ times the number of draw calls compared to `OpenGL`. `Metal`, however, is not cross-platform as `OpenGL` is, so it is not meant to be a replacement for `OpenGL`.
- allows to also run `compute` applications with performance levels comparable to similar technologies such as `CUDA` or `OpenCL`.
- has custom shader language that allows shaders precompiling so they are a lot faster at run time. 
- has built-in memory and resource management, particularized to these platforms.

Until next time!