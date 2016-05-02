---
published: false
title: Using MetalKit part 10
layout: post
---
Today we will look at the only other type of `shader function` we have not used so far, the __kernel function__ or __compute shader__. You will seldom hear even a variation of intermixing words from both of them. A few important facts to keep in mind about them: there is no rendering going on, the function always returns `void` and its name always starts with the __kernel__ keyword, just like the other functions we used before we preceded by the `vertex` and `fragment` keyword.

Let's start by stripping down the playground we used in [Part 8](http://mhorga.org/2016/03/07/using-metalkit-part-8.html). 

{% highlight swift %} 
{% endhighlight %}

If you are showing the `Timeline` in the `Assistant editor` you should have a similar view:

![alt text](https://github.com/Swiftor/Metal/raw/master/images/chapter10_1.png "1")

Special thanks to [Chris Wood](https://twitter.com/_psonice) for his valuable advising. The [source code](https://github.com/Swiftor/Metal/tree/master/ch10) is posted on Github as usual.

Until next time!