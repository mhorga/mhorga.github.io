---
published: false
title: Ray tracing in a Swift playground part 4
layout: post
---
Let's continue working on our `ray tracer` and pick up where we left off last week. First, as you are already used to, we'll do more code cleaning. I went ahead and replaced all classes with structs, and also used proper naming conventions (such as capitalization of types) this time. You can see the modified code in this week's repository. To keep this article short, I am not going over the cleaning procedure this time, but you will notice the transformations were rather minimal.

Last time we looked at how `lambertian` and `metal` materials are rendered. The last type of material we need to look into is called __dielectric__ and you can recognize it when looking at water or at objects made of glass. When `dielectrics` are hit by a ray, the ray splits in two: a `reflected` (bounced) ray and a `refracted` (propagated) ray. `Refraction` is described by [Snell's law](https://en.wikipedia.org/wiki/Snell%27s_law). Let's see how this law translates into code. In `material.swift` create a __refract()__ function:

{% highlight swift %}
func refract(v: float3, n: float3, ni_over_nt: Float) -> float3? {
    let uv = normalize(v)
    let dt = dot(uv, n)
    let discriminant = 1.0 - ni_over_nt * ni_over_nt * (1.0 - dt * dt)
    if discriminant > 0 {
        return ni_over_nt * (uv - n * dt) - n * sqrt(discriminant)
    }
    return nil
}
{% endhighlight %}

Next, let's create the __Dielectric__ struct. Notice that `attenuation` is always __1__ because `dielectrics` never absorb anything from an incident ray:

{% highlight swift %}
struct Dielectric: Material {
    var ref_index: Float = 1
    
    func scatter(ray_in: Ray, _ rec: Hit_record, inout _ attenuation: float3, inout _ scattered: Ray) -> Bool {
        var ni_over_nt: Float = 1
        var outward_normal = float3()
        let reflected = reflect(ray_in.direction, n: rec.normal)
        attenuation = float3(1, 1, 1)
        if dot(ray_in.direction, rec.normal) > 0 {
            outward_normal = -rec.normal
            ni_over_nt = ref_index
        } else {
            outward_normal = rec.normal
            ni_over_nt = 1 / ref_index
        }
        let refracted = refract(ray_in.direction, n: outward_normal, ni_over_nt: ni_over_nt)
        if refracted != nil {
            scattered = Ray(origin: rec.p, direction: refracted!)
        } else {
            scattered = Ray(origin: rec.p, direction: reflected)
            return false
        }
        return true
    }
}
{% endhighlight %}

We first compute an outward normal depending on whether the dot product between the ray and the hit point is positive or not, and then we use that to compute the refracted ray. In case it comes out `nil`, we reflect the ray, otherwise we refract it. In `pixel.swift` replace the second `metal` sphere with a `dielectric` one:

{% highlight swift %}
object = sphere(c: float3(x: -1, y: 0, z: -1), r: 0.5, m: Dielectric())
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing8.png "Raytracing 8")

Glass surfaces have reflectivity that varies with the angle. When you look at it perpendicularly, the reflectivity is the lowest possible, if any. The smaller the viewing angle gets, the higher the reflectivity is, and other objects from the world are mirrored more clearly by the glass surface. This effect can be computed with the Schlick polynomial approximation:

{% highlight swift %}
func schlick(cosine: Float, _ index: Float) -> Float {
    var r0 = (1 - index) / (1 + index)
    r0 = r0 * r0
    return r0 + (1 - r0) * powf(1 - cosine, 5)
}
{% endhighlight %}

The __scatter()__ function needs to be adapted to use this approximation:

{% highlight swift %}
func scatter(ray_in: Ray, _ rec: Hit_record, inout _ attenuation: float3, inout _ scattered: Ray) -> Bool {
    var reflect_prob: Float = 1
    var cosine: Float = 1
    var ni_over_nt: Float = 1
    var outward_normal = float3()
    let reflected = reflect(ray_in.direction, n: rec.normal)
    attenuation = float3(1, 1, 1)
    if dot(ray_in.direction, rec.normal) > 0 {
        outward_normal = -rec.normal
        ni_over_nt = ref_index
        cosine = ref_index * dot(ray_in.direction, rec.normal) / length(ray_in.direction)
    } else {
        outward_normal = rec.normal
        ni_over_nt = 1 / ref_index
        cosine = -dot(ray_in.direction, rec.normal) / length(ray_in.direction)
    }
    let refracted = refract(ray_in.direction, n: outward_normal, ni_over_nt: ni_over_nt)
    if refracted != nil {
        reflect_prob = schlick(cosine, ref_index)
    } else {
        scattered = Ray(origin: rec.p, direction: reflected)
        reflect_prob = 1.0
    }
    if Float(drand48()) < reflect_prob {
        scattered = Ray(origin: rec.p, direction: reflected)
    } else {
        scattered = Ray(origin: rec.p, direction: refracted!)
    }
    return true
}
{% endhighlight %}

Note that we are now refracting based on a reflectivity threshold we set to __1__. There is an easy way (trick) to get a hollow glass surface. If the radius is negative, even though the geometry remains unaffected, the normal will point inward and the result will be a nice looking hollow glass sphere. Let's add one more `dielectric` sphere with a negative radius:

{% highlight swift %}
object = sphere(c: float3(x: -1, y: 0, z: -1), r: -0.49, m: Dielectric())
{% endhighlight %}

You should be able to see the hollow glass sphere now. Before wrapping up, we need to do one more thing - fix the camera, so we can look at objects from different angles and distances. First, we need a `field of view` for our camera. Then, we need a `lookFrom` point and a `lookAt` point to set the direction our camera will look. Finally, we need an `up` vector so we can rotate the camera around its direction, and always know where `up` is. In `ray.swift` let's replace our old camera with this one:

{% highlight swift %}
struct Camera {
    let lower_left_corner, horizontal, vertical, origin, u, v, w: float3
    var lens_radius: Float = 0.0
    init(lookFrom: float3, lookAt: float3, vup: float3, vfov: Float, aspect: Float) {
        let theta = vfov * Float(M_PI) / 180
        let half_height = tan(theta / 2)
        let half_width = aspect * half_height
        origin = lookFrom
        w = normalize(lookFrom - lookAt)
        u = normalize(cross(vup, w))
        v = cross(w, u)
        lower_left_corner = origin - half_width * u - half_height * v - w
        horizontal = 2 * half_width * u
        vertical = 2 * half_height * v
    }
    func get_ray(s: Float, _ t: Float) -> Ray {
        return Ray(origin: origin, direction: lower_left_corner + s * horizontal + t * vertical - origin)
    }
}
{% endhighlight %}

In `pixel.swift` replace the line where we make a call to the camera, with this code:

{% highlight swift %}
let lookFrom = float3(0, 1, -4)
let lookAt = float3()
let vup = float3(0, -1, 0)
let cam = Camera(lookFrom: lookFrom, lookAt: lookAt, vup: vup, vfov: 50, aspect: Float(width) / Float(height))
{% endhighlight %}

In the main playground page, see the generated new image:

![alt text](https://github.com/mhorga/mhorga.github.io/raw/master/images/raytracing9.png "Raytracing 9")

The [source code](https://github.com/Swiftor/Raytracing4) is posted on Github as usual.

Until next time!