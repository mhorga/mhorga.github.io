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

{% highlight swift %}
// declare the audio player as a property
var audioPlayer: AVAudioPlayer!
// inside viewDidLoad initialize
if let filePath = NSBundle.mainBundle().pathForResource("myAudioFile", ofType: "mp3") {
    let url = NSURL.fileURLWithPath(filePath)
    audioPlayer = AVAudioPlayer(contentsOfURL: url, error: nil)
}
// in another method (e.g. an action) play the audio file
audioPlayer.play()
{% endhighlight %}

Now you can apply various effects on the audio file, such as increasing/decreasing the speed or the pitch.