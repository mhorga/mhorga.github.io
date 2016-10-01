---
published: true
title: Markup for Xcode playgrounds
layout: post
---
Today we will look into authoring features offered by `Xcode` to make our playgrounds richer in content. To start, let's create a new playground. Inside, type the following code:

{% highlight swift %}
/*:
# Hello playground!

A *simple* demo with _markup_ examples.
*/

//: Another **variable** of type __String__

//: [Next](@next)
{% endhighlight %}

I know! What's with all this non sense? Well, let's take it easy and digest the syntax. First, we have a comment block marked by a starting __/\*__ symbol and an ending __\*/__ symbol. Nothing new here. What's special though is when we  added an extra __:__ symbol at the beginning of the block. This will make it special later on when we turn on the `markup` validator.

Inside the block we have a __#__ symbol that gives us a __level 1 heading__ (largest possible). Next, we have the word `simple` surrounded by __\*__ symbols. This will give our word the `italic` formatting. Then we have the __\___ symbol as another way of having `italic` formatting. Going further, we see two ways to have `bold` formatting, either by using __\**__ or __\____ symbols to surround our text.

Finally, we notice on the last line that the word __Next__ is first surrounded by square brackets and then preceded by the __@__ symbol and surrounded by parentheses. This is a composed syntax expression. You will later learn that this syntax is used for `URLs`, `images` and page `navigation`. In this case we have navigation to the next page available in the playground. Let's see how our page looks so far. For this, let's activate the `markup` validator. On the `Xcode` menu, go to `Editor` --> `Show Rendered Markup`:

![alt text](https://github.com/Swiftor/Markup/raw/master/images/page1.png "Page 1")

As of now, there is no other page in our playground (even though the image shows there are :-D) so let's create one first. At the bottom left side of the playground there is a __+__ button. Click it to add a __New Page__ to the playground. Name it __page2__ or whatever name you wish. If you go back to the first page and click `Next` it will go to your (empty) second page. Everything looks great so far! Next, let's go back to the `Editor` menu and choose `Show Raw Markup` so we can write code again. On the second page let's write the following markup code:

{% highlight swift %}
//: [Previous](@previous)

//: This is also a Level 1 Heading
//: =========================
//: ## This is a Level 2 Heading
//: ### This is a Level 3 Heading

/*: 
## Countries
> 1. Brazil
> 2. Vietnam
> 3. Colombia

(the > symbol denotes a new section)
*/

/*: 
## Points to Remember
* Empty lines end the single line comment delimiter block
* Comment content ends at a newline
* Commands that work in a comment block work in single line
     * This **includes** text formatting commands
*/

//: [Next](@next)
{% endhighlight %}

You will immediately recognize what the first line does, it goes back the previous page, as you guessed. The next couple of lines are interesting. Everything you accompany on the next line with __=__ symbols will be formatted as __level 1 heading__, so this is just another way to do it, besides using the __#__ symbol in front of the line. Likewise, you can have smaller level headings by using  __##__ or __###__.

Next, we have two ways of presenting lists, `ordered` and `unordered`. One thing worth mentioning is the section delimiter line we can get by using the __>__ symbol. Also, by using different levels of indentation we can have different symbols for presenting the lower level lists. Let's see how our page looks:

![alt text](https://github.com/Swiftor/Markup/raw/master/images/page2.png "Page 2")

Let's create another page and name it __page3__, for example. On the second page click `Next` so we can write the following content on our new page:

{% highlight swift %}
//: [Previous](@previous)

/*:
This text is above the horizontal rule

---
And this is below
*/

/*: Setup and use a link reference.
[The Swift Programming Language]: http://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/ "Some hover text"

For more information, see [The Swift Programming Language].
*/

//: show Swift keywords such as `for` and `let` as monspaced font.

/*:
\* This is not a bullet item
* but this is a bullet item
*/

//: [Next](@next)
{% endhighlight %}

The next new syntax element we notice is another way of having section delimiters, or horizontal rules, by using __---__ (3 hyphen) symbols. Next, we see how to reference `URLs` as clickable links. Also, once you defined a link you can reuse it later on in your page by simply referencing its name as you see on the next line above. The next syntax element is the `back tick` (reversed apostrophe) which allows us to type keywords with an emphasised font. We can also use `\` backslash to escape elements that we want to have a different result, such as a `*` not making one of our lines a bullet point. Let's see how our page looks so far:

![alt text](https://github.com/Swiftor/Markup/raw/master/images/page3.png "Page 3")

Finally, let's create a last page named __page4__ or so, and write this code on it:

{% highlight swift %}
//: [Previous](@previous)

//: ![Icon for a playground](http://devimages.apple.com.edgekey.net/swift/images/playgrounds.png "A playground image")

//: [go back to first page](page1)
{% endhighlight %}

Similar to the way we referenced links, we can also reference `images`. The only difference is that we use the exclamation point `!` in front of the image name, and that we can give it an `Alt` name for accessibility reasons. To wrap up this markup syntax overview, we only look at one more element, how to jump all the way to the starting page of the playground, using the same navigation syntax, of course. You need to give it the exact name of the page in parentheses or the navigation will not work. Let's see how our page looks:

![alt text](https://github.com/Swiftor/Markup/raw/master/images/page4.png "Page 4")

The playground is available on [Github](https://github.com/mhorga/Markup/).

Until next time!