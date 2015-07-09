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

// inside viewDidLoad initialize it
let filePath = NSBundle.mainBundle().pathForResource("myAudioFile", ofType: "mp3")
let url = NSURL.fileURLWithPath(filePath)
audioPlayer = AVAudioPlayer(contentsOfURL: url, error: nil)

// inside the calling/action method play the audio file
audioPlayer.play()
{% endhighlight %}

Now you could apply various effects on the audio file, such as increasing/decreasing the _speed_ or the _pitch_. For changing the playback speed you need to first enable the audio player's __rate__ property. Then inside your calling method you can set the rate to a value between __0.5__ (slowest) and __2.0__ (fastest). A value of __1.0__ would be the normal speed, so you can play the file at any speed from half the normal speed to double the normal speed.

{% highlight swift %} 
// right after initializing audioPlayer, enable its rate
audioPlayer.enableRate = true

// before playing the audio file, change its rate
audioPlayer.rate = 0.5
{% endhighlight %}

To change the pitch of the audio file, the following steps need to take place in order:

- create an AVAudioEngine object
- create an AVAudioPlayerNode object
- attach AVAudioPlayerNode to AVAudioEngine
- create an AVAudioUnitTimePitch object
- attach AVAudioUnitTimePitch to AVAudioEngine
- connect AVAudioPlayerNode to AVAudioUnitTimePitch
- connect AVAudioUnitTimePitch to an output
- play the audio file

The pitch is measured in _cents_, a logarithmic value used for measuring musical intervals. One octave equals 1200 cents. One musical semitone is equal to 100 cents. The default value is 1.0 (normal pitch). The range of values is -2400 (lowest pitch) to 2400 (highest pitch). Here is how this could translate into code:

{% highlight swift %}
// declare the audio engine as a property 
var audioEngine: AVAudioEngine!

// inside viewDidLoad initialize it
audioEngine = AVAudioEngine()

// inside the calling/action method increase the pitch
var audioPlayerNode = AVAudioPlayerNode()
audioEngine.attachNode(audioPlayerNode)
var changePitchEffect = AVAudioUnitTimePitch()
changePitchEffect.pitch = 1000
audioEngine.attachNode(changePitchEffect)
audioEngine.connect(audioPlayerNode, to: changePitchEffect, format: nil)
audioEngine.connect(changePitchEffect, to: audioEngine.outputNode, format: nil)
audioPlayerNode.play()
{% endhighlight %}

We can also use the iPhone's microphone to record an audio file that we can play, instead of the sample file we added to our Xcode project. Here is how we could do it:

{% highlight swift %}
// declare the audio recorder as a property
var audioRecorder:AVAudioRecorder!

// inside the calling/action method record an audio file
let dirPath = NSSearchPathForDirectoriesInDomains(.DocumentDirectory, .UserDomainMask, true)[0] as! String
let recordingName = "my_audio.wav"
let pathArray = [dirPath, recordingName]
let filePath = NSURL.fileURLWithPathComponents(pathArray)
var session = AVAudioSession.sharedInstance()
session.setCategory(AVAudioSessionCategoryPlayAndRecord, error: nil)
audioRecorder = AVAudioRecorder(URL: filePath, settings: nil, error: nil)
audioRecorder.meteringEnabled = true
audioRecorder.prepareToRecord()
audioRecorder.record()

// inside another method that stops the recording
audioRecorder.stop()
var audioSession = AVAudioSession.sharedInstance()
audioSession.setActive(false, error: nil)
{% endhighlight %}

Now we can replace the hardcoded file (myAudioFile.mp3) with our recording stored in the __recordingName__ variable. Every time the recording is started, this file will be overwritten which is useful to saving space. You can see [my demo app](https://github.com/mhorga/PitchPerfect) (based on Udacity's course) using all these concepts.

Until next time!