---
published: false
title: Audio processing with AVFoundation
layout: post
---
In order to be able to play audio files inside an iOS app, you first need to add the audio file (e.g. myAudioFile.mp3) to your Xcode project, and then follow the steps below:

- create a path to the audio file
- create an instance of AVAudioPlayer
- play the audio file

Here is how this could translate into code:

> // declare the audio player as a property<br />
var audioPlayer: AVAudioPlayer!

> // initialize the audio player<br />
let filePath = NSBundle.mainBundle().pathForResource("myAudioFile", ofType: "mp3")<br />
let url = NSURL.fileURLWithPath(filePath)<br />
audioPlayer = AVAudioPlayer(contentsOfURL: url, error: nil)

> // play the audio file<br/>
audioPlayer.play()

Now you can apply various effects on the audio file, such as increasing/decreasing the speed or the pitch. For changing the playback speed you need to first enable the audio player's __rate__ property:

> // right after initializing audioPlayer, enable the rate<br />
audioPlayer.enableRate = true

Then inside your calling method you can set the rate to a value between __0.5__ (slowest) and __2.0__ (fastest). A value of __1.0__ would be the normal speed, so you can play the file at any speed from half the normal speed to double the normal speed.

> // before playing the audio file change the rate<br />
audioPlayer.rate = 0.5

To change the pitch of the audio file, the following steps need to take place in order:

- create an AVAudioEngine object
- create an AVAudioPlayerNode object
- attach AVAudioPlayerNode to AVAudioEngine
- create an AVAudioUnitTimePitch object
- attach AVAudioUnitTimePitch to AVAudioEngine
- connect AVAudioPlayerNode to AVAudioUnitTimePitch
- connect AVAudioUnitTimePitch to an output

{% highlight swift %}

{% endhighlight %}

You can see [my demo app](https://github.com/mhorga/PitchPerfect) (based on Udacity's course) using all these concepts.

Until next time!