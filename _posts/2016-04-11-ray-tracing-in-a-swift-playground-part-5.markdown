---
published: true
title: Ray tracing in a Swift playground part 5
layout: post
---
Let's continue working on our `ray tracer` and pick up where we left off last week. Now that we know how to generate spheres of different materials and know how to look at them from different angles, let's see how we can generate way more than just __3__ spheres. 

In `pixel.swift` let's create a new __random_scene()__ method:

{% highlight swift %}
func random_scene() -> Hitable_list {
    var objects = [Hitable]()
    objects.append(Sphere(c: float3(0, -1000, 0), r: 1000, m: Lambertian(albedo: float3(0.5, 0.5, 0.5))))
    for a in -2..<3 {
        for b in -2..<3 {
            let materialChoice = drand48()
            let center = float3(Float(a) + 0.9 * Float(drand48()), 0.2, Float(b) + 0.9 * Float(drand48()))
            if length(center - float3(4, 0.2, 0)) > 0.9 {
                if materialChoice < 0.8 {   // diffuse
                    let albedo = float3(Float(drand48()) * Float(drand48()), Float(drand48()) * Float(drand48()), Float(drand48()) * Float(drand48()))
                    objects.append(Sphere(c: center, r: 0.2, m: Lambertian(albedo: albedo)))
                } else if materialChoice < 0.95 {   // metal
                    let albedo = float3(0.5 * (1 + Float(drand48())), 0.5 * (1 + Float(drand48())), 0.5 * (1 + Float(drand48())))
                    objects.append(Sphere(c: center, r: 0.2, m: Metal(albedo: albedo, fuzz: Float(0.5 * drand48()))))
                } else {    // glass
                    objects.append(Sphere(c: center, r: 0.2, m: Dielectric()))
                }
            }
        }
    }
    objects.append(Sphere(c: float3(0, 0.7, 0), r: 0.7, m: Dielectric()))
    objects.append(Sphere(c: float3(-3, 0.7, 0), r: 0.7, m: Lambertian(albedo: float3(0.4, 0.2, 0.1))))
    objects.append(Sphere(c: float3(3, 0.7, 0), r: 0.7, m: Metal(albedo: float3(0.7, 0.6, 0.5), fuzz: 0.0)))
    return Hitable_list(list: objects)
}
{% endhighlight %}

This method generates roughly __25__ smaller spheres and assigns a material to each of them based on a random number that determines whether the sphere material will be `lambertian`, `metal` or `glass`. Then we add each sphere to a list that the function needs to return. We also add the big sphere and the __3__ initial smaller spheres we used to have.

Then inside the __imageFromPixels()__ method we replace the code block where we used to add the spheres:

{% highlight swift %}
var objects = [Hitable]()
var object = Sphere(c: float3(0, -100.5, -1), r: 100, m: Lambertian(albedo: float3(0.7, 0.23, 0.12)))
objects.append(object)
object = Sphere(c: float3(1, 0, -1), r: 0.5, m: Metal(albedo: float3(0.8, 0.6, 0.2), fuzz: 0.1))
objects.append(object)
object = Sphere(c: float3(-1, 0, -1), r: 0.5, m: Dielectric())
objects.append(object)
object = Sphere(c: float3(-1, 0, -1), r: -0.49, m: Dielectric())
objects.append(object)
object = Sphere(c: float3(0, 0, -1), r: 0.5, m: Lambertian(albedo: float3(0.24, 0.5, 0.15)))
objects.append(object)
let world = Hitable_list(list: objects)
{% endhighlight %}

with a single line that creates the random world instead:

{% highlight swift %}
let world = random_scene()
{% endhighlight %}

I would normally tell you to try rendering the scene __now__, but there is a great speed up hint I learned from [hyperjeff](https://twitter.com/hyperjeff) and which allows us to get images of greater quality, much faster. Still inside the __imageFromPixels()__ method, replace the outer loop first line:

{% highlight swift %}
for i in 0..<width {
{% endhighlight %}

with this block of code:

{% highlight swift %}
let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
dispatch_apply(width, queue) { i in
{% endhighlight %}

By using `Grand Central Dispatch` threading, the rendering finishes __3__ times faster! In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing.png "Raytracing")

This image was generated using a value of __ns = 50__, a sphere generator range from __-7..<7__ and an image resolution of __800 x 400__. The rendering took __752__ seconds to run so if you want a quick, __5__-second rendering I suggest using a value of __ns = 10__, a sphere generator range from __-2..<3__ and an image resolution of __400 x 200__. The [source code](https://github.com/mhorga/Raytracing5) is posted on Github as usual.

Until next time!