---
published: true
title: iOS persistence with Core Data
layout: post
---
In the other two articles from the `iOS Persistence` series we looked at `NSUserDefaults` and then at the `NSKeyerArchiver` as ways to persist user data. This time we will use `Core Data` for persistence. Let’s create another `Master-Detail` project but this time make sure to enable __Use CoreData__ before clicking `Next` and `Create`. Now run the project, add a new row from the __+__ button, stop the app and run it again. You will notice the data persisted! End of lesson. 

You didn’t hope we would just stop here, did you? What if we have an existing project that did not use Core Data before, but we want to implement Core Data into it now. We will go step by step and show how we can transform an existing project into a Core Data-powered project. So let’s create the `Master-Detail` project again but do __not__ enable `Use CoreData` this time. We are going to make the conversion in stages:

1. Add the Core Data Stack
2. Add a Core Data Model
3. Make entity subclasses of NSManagedObject
4. Inserting objects into the context
5. Saving the context
6. Fetching objects out of the context

1) We start by adding a Core Data stack to the project. Let’s create a class intuitively named, __CoreDataStack__. A core data stack, as you can see in the diagram below, consists of a context, a model, a coordinator and a store.

![alt text](https://developer.apple.com/library/ios/documentation/DataManagement/Devpedia-CoreData/Art/single_persistent_stack.jpg "Core Data stack")

In order to make sure we cannot access the core data stack in two separate instances, we create a `singleton` which guarantees that it will be the only handle we have over this core data stack. The class skeleton looks like this:

{% highlight swift %}
import CoreData

private let SQLITE_FILE_NAME = "CoreData.sqlite"

class CoreDataStack {
    
    class func sharedInstance() -> CoreDataStack {
        struct Static {
            static let instance = CoreDataStack()
        }
        return Static.instance
    }
}
{% endhighlight %}

First we imported `CoreData`, then we declared the name of the file where our data will be stored on the disk, and finally we created the `singleton`. Next we add four `lazy properties` - which are only calculated the first time they are used. The first one we add is __applicationDocumentsDirectory__:

{% highlight swift %}
lazy var applicationDocumentsDirectory: NSURL = {
    println("Instantiating the applicationDocumentsDirectory property")
    let urls = NSFileManager.defaultManager().URLsForDirectory(.DocumentDirectory, inDomains: .UserDomainMask)
    return urls[urls.count-1] as! NSURL
}()
{% endhighlight %}

In Core Data, the documents directory holds the .sqlite database file that stores our data. The next property is __managedObjectContext__:

{% highlight swift %}
lazy var managedObjectContext: NSManagedObjectContext? = {
    println("Instantiating the managedObjectContext property")
    let coordinator = self.persistentStoreCoordinator
    if coordinator == nil {
        return nil
    }
    var managedObjectContext = NSManagedObjectContext()
    managedObjectContext.persistentStoreCoordinator = coordinator
    return managedObjectContext
}()
{% endhighlight %}

This property is used in Core Data to get a reference to the shared context for fetching and inserting objects. The next property is __managedObjectModel__ which is an object that knows about our model classes, including their properties and relationships. 
{% highlight swift %}
lazy var managedObjectModel: NSManagedObjectModel = {
    println("Instantiating the managedObjectModel property")
    let modelURL = NSBundle.mainBundle().URLForResource("Model", withExtension: "momd")!
    return NSManagedObjectModel(contentsOfURL: modelURL)!
}()
{% endhighlight %}

Finally, the fourth property is __persistentStoreCoordinator__ which is an object that handles the file management for the context and it hides the complexity of dealing with a `SQLite` file. 
{% highlight swift %}
lazy var persistentStoreCoordinator: NSPersistentStoreCoordinator? = {
    println("Instantiating the persistentStoreCoordinator property")
    var coordinator: NSPersistentStoreCoordinator? = NSPersistentStoreCoordinator(managedObjectModel: self.managedObjectModel)
    let url = self.applicationDocumentsDirectory.URLByAppendingPathComponent(SQLITE_FILE_NAME)
    println("sqlite path: \(url.path!)")
    var error: NSError? = nil
    if coordinator!.addPersistentStoreWithType(NSSQLiteStoreType, configuration: nil, URL: url, options: nil, error: &error) == nil {
        coordinator = nil
        let dict = NSMutableDictionary()
        dict[NSLocalizedDescriptionKey] = "Failed to initialize the application's saved data"
        dict[NSLocalizedFailureReasonErrorKey] = "There was an error creating or loading the application's saved data."
        dict[NSUnderlyingErrorKey] = error
        error = NSError(domain: "YOUR_ERROR_DOMAIN", code: 9999, userInfo: dict as [NSObject : AnyObject])
        NSLog("Unresolved error \(error), \(error!.userInfo)")
        abort()
    }
    return coordinator
}()
{% endhighlight %}

2) Now that we are done with configuring the `Core Data` stack, we need to proceed to the next step, add a `Core Data Model`. For this we will create an entity to hold information about our model. Let’s create the data model file in `Xcode`. From the `File` menu, choose `New File` and under `Core Data` choose `Data Model`. Leave its default name and click `Create`. You will notice a new file __Model.xcdatamodeld__ appears in our project now. 

Go to this new file so we can create our entity. Click the __Add Entity__ button to create the entity and name it __Event__. Under `Attributes` click the __+__ button to add an attribute named __timeStamp__ of type `Date`. One more thing to do, select `Event` and open the `Utilities` panel. In the `Data Model Inspector` under `Class` change the name to `Event` instead of the default name `Xcode` gave it. If at this point (or any point in configuring Core Data) you get the message bellow, simply delete the app in the simulator and run it again:

{% highlight swift %}
The model used to open the store is incompatible with the one used to create the store
{% endhighlight %}

Let’s finally use the Core Data stack in our `MasterViewController` class (don’t forget to import CoreData):

{% highlight swift %}
var sharedContext: NSManagedObjectContext {
    return CoreDataStack.sharedInstance().managedObjectContext!
}
{% endhighlight %}

Now you can access it inside the `viewDidLoad` method like this:

{% highlight swift %}
print(sharedContext)
{% endhighlight %}

Now you can see the proper stack initialization:

{% highlight swift %}
Instantiating the managedObjectContext property
Instantiating the persistentStoreCoordinator property
Instantiating the managedObjectModel property
Instantiating the applicationDocumentsDirectory property
sqlite path: /Users/.../Documents/CoreData.sqlite
<NSManagedObjectContext: 0x7f84f87078d0>
{% endhighlight %}

Here is the reason it initializes in this order. The real purpose of this stack is to supply a `Context` - the one we use to fetch and insert managed objects. We begin the initialization of the lazy properties by asking for the `managedObjectContext`. The context needs an assistant object that will abstract away all of the details of how data is stored on the hard drive. This assistant object is the `Persistent Store Coordinator`. It manages the `SQLite` file that persists the data, so the context instantiates the `persistentStoreCoordinator` property next. The persistent store coordinator needs an `Object Model` that knows about our entity objects, and a path to the `SQLite` file in the documents directory. While the persistent store coordinator is being made, it instantiates the `managedObjectModel`. Then the `applicationDocumentsDirectory` is initialized in the course of creating the persistent store coordinator and that successfully created the whole stack.

3) This entity we created will map each new row we create when we click on the __+__ right side bar button item and the attribute will represent its time stamp (date). Now we need to make this entity a `NSManagedObject` so first select the `CoreData.xcdatamodeld` file, then on the `Xcode` menu under `Editor` select the `Create NSManagedObject Subclass` option. Make sure to check `Model` and then `Event` on the next screen before you click `Create`. You will notice a class named `Event` was created. The entire class should look like this:

{% highlight swift %}
import CoreData

@objc(Event)
class Event: NSManagedObject {

    @NSManaged var timeStamp: NSDate

    override init(entity: NSEntityDescription, insertIntoManagedObjectContext context: NSManagedObjectContext?) {
        super.init(entity: entity, insertIntoManagedObjectContext: context)
    }
    
    init(context: NSManagedObjectContext) {
        let entity =  NSEntityDescription.entityForName("Event", inManagedObjectContext: context)!
        super.init(entity: entity, insertIntoManagedObjectContext: context)
        timeStamp = NSDate()
    }
}
{% endhighlight %}

You will notice a strange line `@objc(Event)`. This makes the `Event` visible to the `Core Data` code. Notice that both the class (entity) and the attribute are `NSManaged` by `Core Data`. Next, we notice two `init` methods. The first one is the standard `Core Data` initializer method, and the second one is a new method we build that only needs a context.

4) In order to be able to insert objects into the context we first need to change the `objects` array type from `AnyObject` to `Event`:

{% highlight swift %}
var objects: [Event]!
{% endhighlight %}

Since we now have a different array type let’s update the line below in `prepareForSegue` and `cellForRowAtIndexPath` methods:

{% highlight swift %}
let object = objects[indexPath.row] as Event
{% endhighlight %}

The next line we need to change is inside the `insertNewObject` method:

{% highlight swift %}
objects.insert(Event(context: sharedContext), atIndex: 0)
{% endhighlight %}

5) Next, we need to save the context after inserting new objects in our array. For this we need to add a new method to our `CoreDataStack` class:

{% highlight swift %}
func saveContext () {
    if let context = self.managedObjectContext {
        var error: NSError? = nil
        if context.hasChanges && !context.save(&error) {
            NSLog("Unresolved error \(error), \(error!.userInfo)")
            abort()
        }
    }
}
{% endhighlight %}

We can call this class in `MasterViewController` every time a new event is created in the `insertNewObject` method:

{% highlight swift %}
CoreDataStack.sharedInstance().saveContext()
{% endhighlight %}

6) Finally, we need to fetch objects out of the context. For this we create a new method:

{% highlight swift %}
func fetchAllEvents() -> [Event] {
    var error: NSError?
    let fetchRequest = NSFetchRequest(entityName: "Event")
    let results = sharedContext.executeFetchRequest(fetchRequest, error: &error)
    if let error = error {
        println("Error while fetching objects: \(error)")
    }
    return results as! [Event]
}
{% endhighlight %}

and then we call it in `viewDidLoad`:

{% highlight swift %}
objects = fetchAllEvents()
{% endhighlight %}

We need to also fix the label text in `cellForRowAtIndexPath`:

{% highlight swift %}
cell.textLabel!.text = object.timeStamp.description
{% endhighlight %}

Now run the project and add a few events. Re-run the project and notice the new events were persisted. As a logical next step, you could implement persistence for when events are deleted from the array.

Until next time!
