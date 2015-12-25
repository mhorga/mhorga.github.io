---
published: true
title: Error handling in Swift
layout: post
---
In this article we will look at how errors are handled in `Swift 2` in such a way that your programs will exit gracefully instead of crashing. If you want to __throw__ an error, the object thrown must conform to the __ErrorType__ protocol. `Enums` are appropriate for classifying errors, so let's create one that provides a way of keep track of all possible errors we could get. Let's assume this time we can only have two errors: 

{% highlight swift %}
enum Error: ErrorType {
    case InvalidData
    case NoConnection
}
{% endhighlight %}

Let's also create a struct named __User__ that holds a person's name. If we believe our struct might fail when being instantiated, we can use a __failable initializer__ to ensure that the struct returns `nil` instead crashing the app. 

{% highlight swift %}
struct User {
    var firstname = ""
    var lastname = ""
    init? (dict: [String : String]) {
        guard let firstname = dict["firstname"], lastname = dict["lastname"] else {return nil}
        self.firstname = firstname
        self.lastname = lastname
    }
}
{% endhighlight %}

Inside the __init(:)__ method we use a __guard__ statement to make sure both __firstname__ and __lastname__ exist so that the initialization is either successful, or otherwise it fails gracefully. 

A function that can throw an error, or calls a function that can throw an error has to be marked with __throws__. After your program throws an error, you will need to handle that error. Let's create a function __parseData(:)__ that returns a __User__ struct if the provided argument is valid, or otherwise `throws` the first of defined errors, the __InvalidData__ type: 

{% highlight swift %}
func parseData(from input: [String : String]) throws -> User {
    guard let firstname = input["firstname"], lastname = input["lastname"] else {
        throw Error.InvalidData
    }
    return User(dict: ["firstname" : firstname, "lastname" : lastname])!
}
{% endhighlight %}

When calling an error-throwing function, we must embed it in a __do__ block. Within the block, we need to __try__ the function and if it fails, we need to __catch__ it and return an error. When we get to the function that's catching and handling the errors, we don't need to include __throws__ in the declaration. Let's create another function named __testParsing(:)__ that calls our error-throwing `parseData(:)` function: 

{% highlight swift %}
func testParsing(dict: [String : String]) -> String {
    var result = ""
    do {
        let item = try parseData(from: dict)
        result = "\(item.firstname) \(item.lastname)"
    } catch Error.InvalidData {
        result = "Invalid data."
    } catch {
        result = "Unknown error."
    }
    return result
}
{% endhighlight %}

If the function doesn't throw an error, we simply return the user's name. If it throws an error, however, we need to catch it and report the appropriate error message. You notice we have a second, generic `catch` statement which is required so that we have an `exhaustive` handling of the error throwing. Next, let's see how this function works with various arguments:

{% highlight swift %}
print(testParsing(["firstname": "Jane", "lastname": "Doe"]))
print(testParsing(["lastname": "Doe"]))
{% endhighlight %}

In the second call, we omitted the `firstname` in purpose to see how output looks like:

{% highlight swift %}
Jane Doe
Invalid data.
{% endhighlight %}

As expected, the output for the first call is correct, while the output for the second call reports the error type we told it to in the `testParsing(:)` function.

We can also chain multiple throwing function calls together knowing that if any link in the chain fails, the rest of the calls won't execute. First let's create a new method named __testConnection(:)__ to handle the second error we defined in the beginning, the __NoConnection__ type:

{% highlight swift %}
func testConnection(response: Int) throws {
    guard response == 200 else {
        throw Error.NoConnection
    }
}
{% endhighlight %}

It's just a dummy function that simulates checking a network connection, and which throws an error when the `http return code` is different than __200__. To test it, let's also create another function named __testThrowing(::)__ that calls both the throwing functions we created so far:

{% highlight swift %}
func testThrowing(dict: [String : String], code: Int) {
    defer {
        print("Testing should be successful, but sometimes errors could be thrown.")
    }
    do {
        try parseData(from: dict)
        try testConnection(code)
        print("All functions completed successfully.")
    } catch {
        print("There is an least one error in the chain!")
    }
}
{% endhighlight %}

You will notice that we used the __defer__ keyword which allows the closure that follows it to execute only when the function ends. The advantage of using `defer` is that the code will execute when the function throws too, so this is a great place to put code that always needs to run, even when errors occur!
Next, let's see how this function works with various arguments:    

{% highlight swift %}
testThrowing(["firstname": "Jane", "lastname": "Doe"], code: 200)
testThrowing(["firstname": "Jane", "lastname": "Doe"], code: 404)
{% endhighlight %}

We first provide correct input for both arguments, and then we provide a bad `http return code` that should lead to a failure, even though the first argument is still correct:

{% highlight swift %}
All functions completed successfully.
Testing should be successful, but sometimes errors could be thrown.
There is an least one error in the chain!
Testing should be successful, but sometimes errors could be thrown.
{% endhighlight %}

As you expect, the first call returns success while the second reports an error and halts the execution of the entire chain of throwing function calls. This is a powerful feature that we can use whenever we want our program to stop running when there is at least one problem in a long decisional chain. The [source code](https://github.com/Swiftor/ErrorHandling) is posted on `Github`, as usual.

Until next time!