---
published: true
title: Swift is now open source!
layout: post
---
Mark this date: `December 3, 2015` - the day `Swift` has become `open source`! It is absolutely amazing that we are now able to look back into the history of `Swift` and see its evolution; for example the [first commit](https://github.com/apple/swift/commit/18844bc65229786b96b89a9fc7739c0fc897905e) was made by who else but `Chris Lattner`, in `July 2010`. There has been a tremendous interest for the `Swift` main repository, with almost __20k__ stars and __2k__ forks in just the first few days. A total of __17__ repositories were made public by `Apple`. There have even been a few pull requests accepted and merged, the most notable one being the removal of the __++__ and __--__ operators, as well as of the C-style __for__ loop syntax, all this in the upcoming `Swift 3.0` version. If that was not enough, the [Perfect](https://www.perfect.org/) project is already using server-side `Swift`, and here is also a [HTTP server](https://github.com/huytd/swift-http) written in `Swift`.

So what have we got? First, a new website as the [Swift](https://swift.org/) home; here you can find the new blog, the `Swift` binaries for `OS X` and `Ubuntu`, a `Getting Started` guide, the `Documentation` page from where you can download the most recent [Swift Programming Language](https://swift.org/documentation/TheSwiftProgrammingLanguage(Swift2.2).epub) ebook (currently the __2.2__ version), among other resources. Second, we got all the source code grouped in no less than __17__ repositories. The most notable ones are:

* [Swift main](https://github.com/apple/swift) which contains the source code for the `Swift` compiler, standard library, and `SourceKit`.
* [Swift evolution](https://github.com/apple/swift-evolution) which contains documents related to the continued evolution of `Swift`, including goals for upcoming releases proposals for changes to and extensions of `Swift`.
* [Swift corelibs](https://github.com/apple/swift-corelibs-foundation) - there are __3__ `corelibs` repositories but the most important one is `Foundation`, which contains the source code for `Foundation`, which provides common functionality for all applications.
* [Swift package manager](https://github.com/apple/swift-package-manager) which contains the source code for `Package Manager` - a tool for managing the distribution of `Swift` code. It’s integrated with the `Swift` build system to automate the process of downloading, compiling, and linking dependencies. 

I will conclude this article with a friendly call to `contributing` to open source code. There are several ways you could get involved:

* the easiest entry way to contributing is to `document` source code - there are plenty of places that need comments and descriptions.
* another easy way is to add more `unit testing` where the code does not have enough/full coverage
* going further to more advanced ways, you can search for __FIXME__ marks inside the code and attempt to fix them.
* also, there are many __NSUnimplemented()__ marks in the `Foundation` code base that you could attempt to implement.
* finally, and this is the hardest part, try to resolve `bugs` from the [Swift issues](https://bugs.swift.org/issues/) list.

Until next time!