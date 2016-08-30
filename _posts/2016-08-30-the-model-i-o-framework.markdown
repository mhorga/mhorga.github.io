---
published: false
title: The Model I/O framework
layout: post
---
__Model I/O__ was introduced in `2015` for `iOS 9` and `OS X 10.11` and it is a framework that helps us create more realistic and interactive graphics. We can use it to import/export `3D` assets, to describe lighting, materials and environments, to bake lights, to subdivide and voxelize meshes, and for physical based rendering. Model I/O easily integrates our assets with our code in various `3D APIs`:

![alt text](modelio_1.png "Model I/O integration")

In order to import an asset we simply do:

{% highlight swift %}let url = NSURL(string: "/Users/YourUsername/Desktop/imported.obj")
let asset = MDLAsset(URL: url!)
{% endhighlight %}

To export an asset we do:

{% highlight swift %}url = NSURL(string: "/Users/YourUsername/Desktop/exported.obj")
asset.exportAssetToURL(url!)
{% endhighlight %}

Model I/O will save both the __.obj__ file and an additional __.mtl__ file that contains information about the object materials, such as in this example:

{% highlight text %}newmtl lambert1
Ns 0.000000
Ka 0.400000 0.400000 0.400000
Kd 0.480000 0.480000 0.480000
Ks 0.496564 0.496564 0.496564
Ni 1.000000
d 1.000000
illum 2
map_Kd diffuse.png
{% endhighlight %}

where the last line represents the name of the texture map file for the `Lambertian` (diffuse) lighting.

{% highlight swift %}
{% endhighlight %}

Until next time!