---
published: true
title: Protocol oriented programming in Swift
layout: post
---
A few months ago I wrote an [article](http://mhorga.org/2015/07/14/protocols-and-delegates.html) that introduced `protocols` as a way to delegate work to other classes. Today I will talk about how we can leverage `protocols` to achieve great programming design. One huge advantage of using `protocols` over `class inheritance` is that inheritance only applies to classes, while `protocols` can be adopted by classes, structs and enums. Starting with `Swift 2`, `protocols` can now also have `extensions`. That means it is now possible to define default behavior for a `protocol`. And since types can conform to multiple `protocols`, think about the benefit of defining default behaviors from multiple `protocols`!

Let’s start by creating a new `Xcode` playground, and defining a struct named __Location__ that has a __coord__ `(x, y)` property and an __init()__ method: 

{% highlight swift %}
struct Location {
    var coord = (0, 0)
    init (x: Int, y: Int) {
        coord.0 = x
        coord.1 = y
    }
}
{% endhighlight %}

Let’s also create an array to hold our __locations__, create an example `location` and add it to the array:

{% highlight swift %}
var locations = [Location]()
var current = Location(x: 0, y: 0)
locations.append(current)
{% endhighlight %}

So far so good! However, if we add the example location again:

{% highlight swift %}
locations.append(current)
print(locations[0].coord) 
print(locations[1].coord)
{% endhighlight %}

The array will store it as well, and we don’t want to have duplicate locations!

{% highlight swift %}
(0, 0)
(0, 0)
{% endhighlight %}

In order to assure uniqueness of our stored locations we need to use another data structure that has such a feature, a `Set`. Delete the last __4__ lines and modify the locations definition like this:

{% highlight swift %}
var locations = Set<Location>()
{% endhighlight %}

You will notice there is an error saying that `Location` does not conform to the `Hashable` protocol. If you `Option + click` on `Set` you will notice its definition requires it to be of type:

{% highlight swift %}
Set<Element : Hashable>
{% endhighlight %}

Let’s make use of a `protocol extension` now, since we mentioned in the beginning how handy it is for such a situation:

{% highlight swift %}
extension Location: Hashable {
}
{% endhighlight %}

You will notice another error saying that the `protocol` requires property __hashValue__ with type `Int`, so let’s create it inside the extension:

{% highlight swift %}
var hashValue: Int {
    return [coord.0.hashValue, coord.1.hashValue].hashValue
}
{% endhighlight %}

We get yet another error message. If you `Option + click` on `Hashable`, you will notice it conforms to another protocol, `Equatable`, and if you click on `Equatable` you will see that when adopting `Equatable`, the __==__ operator is required to be implemented. The standard library provides implementation for _!=_ however, so we don’t need to deal with that. Let’s create the `equality` operator:

{% highlight swift %}
func ==(lhs: Location, rhs: Location) -> Bool {
    return lhs.coord.0 == rhs.coord.0 && lhs.coord.1 == rhs.coord.1
}
{% endhighlight %}

Great! All the errors have gone away now. Let’s see how the `Set` will help us store unique locations. We already have an example `location` created, so let’s just add it. Then increment the `Y` coordinate, and add the modified location twice. Finally also decrement the `X` location and add it twice as well:

{% highlight swift %}
locations.insert(current)
current.coord.1 += 1
locations.insert(current)
locations.insert(current)
current.coord.0 -= 1
locations.insert(current)
locations.insert(current)
{% endhighlight %}

Now let’s print out `Locations` and see what we got stored in there:

{% highlight swift %}
print((locations.first?.coord)!)
locations.removeFirst()
print((locations.first?.coord)!)
locations.removeFirst()
print((locations.first?.coord)!)
{% endhighlight %}

As expected, instead of seeing duplicates, we only have the __3__ locations we’re supposed to have:

{% highlight swift %}
(0, 0)
(-1, 1)
(0, 1)
{% endhighlight %}

This is only a basic example of the power that `protocol extensions` have in `Swift` programming. You noticed how value types can have behaviors added to them by default, or when needed. The project [source code](https://github.com/Swiftor/ProtocolOrientedProgramming) is posted on `Github`, as usual.

Until next time!