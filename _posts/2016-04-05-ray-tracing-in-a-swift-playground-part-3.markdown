---
published: true
title: Ray tracing in a Swift playground part 3
layout: post
---
Since I missed my Monday post, as a punishment, I will write two posts this week! Let's continue working on our `ray tracer` and pick up where we left off last week. If we want to render spheres of different materials, `Peter Shirley` recommends creating an abstract material class that encapsulates behavior. As a computer scientist myself, I couldn't agree more! 

The `material` class will let us produce a scattered ray and calculate how much it was absorbed or attenuated by its reflectance. Let's create a new file named __material.swift__ or any other name of your choice. Inside, let's create a `protocol` which is well suited for an abstract class in `Swift`:

{% highlight swift %}
protocol material {
    func scatter(ray_in: ray, _ rec: hit_record, inout _ attenuation: float3, inout _ scattered: ray) -> Bool
}
{% endhighlight %}

Now that we have the `material` blueprint, we can render our diffuse (`Lambertian`) spheres by using a new class that conforms to the `material` protocol. We give it an attenuation factor, an initializer and implement the `scatter` function from the protocol, of course:

{% highlight swift %}
class lambertian: material {
    var albedo: float3
    init(a: float3) {
        albedo = a
    }
    func scatter(ray_in: ray, _ rec: hit_record, inout _ attenuation: float3, inout _ scattered: ray) -> Bool {
        let target = rec.p + rec.normal + random_in_unit_sphere()
        scattered = ray(origin: rec.p, direction: target - rec.p)
        attenuation = albedo
        return true
    }
}
{% endhighlight %}

For `metallic` materials, the ray is not scattered randomly as for `Lambertian` materials but the ray is rather reflected with the same angle from the normal, except in the other direction. Again, our class has an attenuation factor, an initializer, the `scatter` function and also a `fuzz` factor which we need so our materials can range from a highly reflective surface to an almost not reflective one:

{% highlight swift %}
class metal: material {
    var albedo: float3
    var fuzz: Float
    init(a: float3, f: Float) {
        albedo = a
        if f < 1 {
            fuzz = f
        } else {
            fuzz = 1
        }
    }
    func scatter(ray_in: ray, _ rec: hit_record, inout _ attenuation: float3, inout _ scattered: ray) -> Bool {
        let reflected = reflect(normalize(ray_in.direction), n: rec.normal)
        scattered = ray(origin: rec.p, direction: reflected + fuzz * random_in_unit_sphere())
        attenuation = albedo
        return dot(scattered.direction, rec.normal) > 0
    }
}
{% endhighlight %}

We need to also have a pointer to the `material` color inside the __hit_record__ struct, in the `objects.swift` file. We can update the pointer when we compute the color later:

{% highlight swift %}
var mat_ptr: material
{% endhighlight %}

Next, we need to adapt our __color()__ function inside the `ray.swift` file, to take the `material` pointer into consideration. Notice that we also added a `depth` factor so we can more accurately compute the color by calling the function recursively when the ray hits an object:

{% highlight swift %}
func color(r: ray, _ world: hitable, _ depth: Int) -> float3 {
    var rec = hit_record()
    if world.hit(r, 0.001, Float.infinity, &rec) {
        var scattered = r
        var attenuantion = float3()
        if depth < 50 && rec.mat_ptr.scatter(r, rec, &attenuantion, &scattered) {
            return attenuantion * color(scattered, world, depth + 1)
        } else {
            return float3(x: 0, y: 0, z: 0)
        }
    } else {
        let unit_direction = normalize(r.direction)
        let t = 0.5 * (unit_direction.y + 1)
        return (1.0 - t) * float3(x: 1, y: 1, z: 1) + t * float3(x: 0.5, y: 0.7, z: 1.0)
    }
}
{% endhighlight %}

Finally, in the `pixel.swift` file we can now create the objects using our new `material` classes:

{% highlight swift %}
var object = sphere(c: float3(x: 0, y: -100.5, z: -1), r: 100, m: lambertian(a: float3(x: 0, y: 0.7, z: 0.3)))
world.add(object)
object = sphere(c: float3(x: 1, y: 0, z: -1.1), r: 0.5, m: metal(a: float3(x: 0.8, y: 0.6, z: 0.2), f: 0.7))
world.add(object)
object = sphere(c: float3(x: -1, y: 0, z: -1.1), r: 0.5, m: metal(a: float3(x: 0.8, y: 0.8, z: 0.8), f: 0.1))
world.add(object)
object = sphere(c: float3(x: 0, y: 0, z: -1), r: 0.5, m: lambertian(a: float3(x: 0.3, y: 0, z: 0)))
world.add(object)
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing7.png "Raytracing 7")

Stay tuned for the next part of this series, where we will look at different type of materials and tweak the camera for a better viewing angle, so the two side spheres don't look distorted anymore. The [source code](https://github.com/Swiftor/Raytracing3) is posted on Github as usual.

Until next time!