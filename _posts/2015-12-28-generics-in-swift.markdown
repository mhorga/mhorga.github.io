---
published: true
title: Generics in Swift
layout: post
---
Last time we looked at `error handling in Swift`. In this article we will look at __Generics__ and how they can make use of one set of types to define a new set of types. Let's start with a simple __Car__ struct. `Car` is a __concrete type__.

{% highlight swift %}
struct Car {
    var make: String
}
{% endhighlight %}

On the other hand, we create the struct `Owner` as a __generic type__ and `T` is its __type parameter__:

{% highlight swift %}
struct Owner<T> {}
{% endhighlight %}

Finally, `driver` is a __specialization__ of the generic type.

{% highlight swift %}
let driver = Owner<Car>()
{% endhighlight %}

Having generics allows us to declare array in various ways:

{% highlight swift %}
let array1 = []
let array2 = [Int]()
let array3 = Array<Int>()
{% endhighlight %}

While `array1` is a `NSArray`, both `array2` and `array3` are representing an `Array` of type `Int`, so their syntax is equivalent. 

Let's look at how the `type parameter` can be exploited further:

{% highlight swift %}
struct Owner<T> {
    var name: String
    var firstOwner: T
    var reuseOwner: T
}
{% endhighlight %}

We can have the same type parameter used also as a struct member. Then creating an `Owner` becomes more interesting:

{% highlight swift %}
let me = Owner(name: "John", firstOwner: Car(make: "Audi"), reuseOwner: Car(make: "BMW"))
{% endhighlight %}

Obviously, generics have `type inferral`:

{% highlight swift %}
let array4 = Array(arrayLiteral: 1, 2, 3)   
{% endhighlight %}

In this case `array4` becomes an `Array<Int>` by type inference. 

For dictionaries there is more information offered. A dictionary is a struct defined as `Dictionary<Key : Hashable, Value>`. What follows after __:__ is either a `supertype` or a `protocol` the key conforms to, and it is named __type constraint__. Dictionaries also have type inferral:

{% highlight swift %}
let ints: [Int: String] = [10: "ten"]
let ages = ["John": 39]
{% endhighlight %}

In this case, the `ages` dictionary is inferred to be of `<String, Int>` type.

As you might have expected, `generics` work well on `optionals` too:

{% highlight swift %}
enum Optional<T> {
    case None
    case Some(T)
}
{% endhighlight %}

So in this case we have the __Optional__ type that can either be of __Some__ type, or __nil__. Then we can test which of the two cases we need to deal with, and react accordingly:

{% highlight swift %}
let employed = Optional<String>.Some("programmer")
let unemployed = Optional<String>.None
if let job = employed {
    print(employed)
} else {
    print(unemployed)
}
{% endhighlight %}

Finally, generics can be used with functions as well. You can notice the `type parameters` are the same as the `function arguments`:

{% highlight swift %}
func concatenate<A, B>(a: A, _ b: B) -> String {
    return "\(a)\(b)"
}

concatenate("$", 5)
{% endhighlight %}

The above function call will print out as you expected:

{% highlight text %}
$5
{% endhighlight %}

The [source code](https://github.com/mhorga/Generics) is posted on `Github`, as usual.

Until next time!