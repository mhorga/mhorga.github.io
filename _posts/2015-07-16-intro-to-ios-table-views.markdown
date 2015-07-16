---
published: true
title: Intro to iOS table views
layout: post
---
Start with a new _Single View Application_ project. Go straight to the storyboard and add a table view to the view controller. On top of the table view, add a table view cell. Select the view controller. Under the _Identity Inspector_ make sure the _Class_ is set to _ViewController_. Select the table view cell. Under the _Attributes Inspector_ change _Style_ to _Subtitle_ and the _Identifier_ to _MyCellReuseIdentifier_. You are done with the storyboard configuration now, and it should look like this:

![alt text](https://github.com/mhorga/mhorga.github.io/blob/master/images/project1.png "Storyboard")

Now go to the ViewController file. Let's conform to the UITableViewDataSource protocol so we can use some of its methods. The ViewController class signature needs to look like this:

{% highlight swift %} 
class ViewController: UIViewController, UITableViewDataSource {
{% endhighlight %}

You will immediately notice an error message:

`Type 'ViewController' does not conform to protocol 'UITableViewDataSource'`

This is because the protocol we want to conform to has at least two methods that we are forced to implement. Delete everything inside ViewController, and add a new array with a few cities as values:

{% highlight swift %} 

    let array = ["Chicago", "New York", "San Francisco"]

{% endhighlight %}

Now add the two methods that the _UITableViewDataSource_ protocol needs implemented:

{% highlight swift %} 

    func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return array.count
    }
    
    func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCellWithIdentifier("MyCellReuseIdentifier") as! UITableViewCell
        cell.textLabel?.text = array[indexPath.row]
        cell.detailTextLabel?.text = "One of the largest IT job markets in US."
        return cell
    }

{% endhighlight %}

In the first method, the only notable thing is how the number of rows is returned - as the size of our array. In the second method, we first create our cell as a UITableViewCell type and identified by the string "MyCellReuseIdentifier" which we set in the storyboard at the beginning. What happens next is interesting: each cell's title will be one value from our array, while the cell's subtitle will be always the same string because we set it this way. Now save your work and run the project. It should look like this:

![alt text](https://github.com/mhorga/mhorga.github.io/blob/master/images/simulator1.png "Simulator")

Wasn't it great to be able set up a functional app that handles data in table view cells, in just a couple of minutes?  Suggestions for a more advanced project:

- move the array in a Model class so we can use a proper MVC design pattern
- set a bigger cell height so it can fit more content
- add labels and images to your cell for a richer content
- create a custom table view cell class and move all the cell details into it
- then dequeue a reusable cell as your new cell type instead of UITableViewCell

Until next time!
