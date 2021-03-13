---
layout: post
title: "The Fright Before Christmas: How a MediaPlayer and OBB file nearly ruined my holidays"
excerpt: "A fun retrospective look at a pesky bug that threatened to ruin my holiday season"
categories: [Android, Tools]
comments: true
medium: https://sp4ghetticode.medium.com/the-fright-before-christmas-how-a-mediaplayer-and-obb-file-nearly-ruined-my-holidays-5f0077a5b59f
image:
  feature: /post/xmas.jpg
  credit: Dawson Lovell
  creditlink: https://unsplash.com/photos/oj_H8dvuw8g
---

Hello and welcome to my final post of 2020, a year that has been full of ups and downs, face masks, viruses, and some seriously strange goings-on within an Android project I maintain.

This post is a fun retrospective on an issue faced within the said project, how it was discovered, and the steps taken to ultimately fix it. I hope it'll serve as a guide on how to approach such an issue (or not) and also provide you with some useful tips to avoid these issues in your own projects. 

For the sake of clarity, the views expressed in this blog post are my own and do not necessarily reflect those of my employer.

# The Background

Of course, in order to understand what went wrong, you need the background. The application in question is a legacy app that I maintain as part of my day job. Its codebase is certainly not modern by today's Android standards but was built fairly solidly during Android 4.x's releases in the early-mid 2010s. I won't go into too much more detail here, purely to avoid naming the client outright, but the app has a solid user base and has been incredibly popular on the Play Store since it was released. All fairly standard stuff for the older apps I tend to work on.

There is one noteworthy aspect of this particular app however, it has *a lot* of in-app audio. How much is a lot? Try 80MB. 

Now you might think 80MB doesn't sound a lot in 2020, but this was 160% of the 50MB size allowance for an APK at the time of the initial release. You must also take into account that this 80MB does not even account for any of the app's code, image assets, or anything else bundled into the app. Adding all of this together, it was easily topping 120MB even with minification. We were _way_ over the hard limit set by the Play Store, but thankfully Google had already thought about scenarios such as this...

# The humble OBB file 📦

In a world in which the [Android App Bundle](https://developer.android.com/guide/app-bundle) did not yet exist, Google's suggested solution to the issue of exceeded the hard-limit for APKs was the 'APK Expansion file', delivered through the Play Store in an ['OBB file format'](https://developer.android.com/google/play/expansion-files).

Put simply, an OBB file is an archive file that the Google Play Store hosts and serves to users once they have downloaded your mobile application. It is saved to the device's shared storage, allowing its content to be accessible from your application once available.

The OBB format is equivalent to that of a ZIP file and allows for a complex file structure with multiple files to be contained within it. For my application, this consisted of several directory trees with a plethora of MPEG-4 audio files spread across them.

However, in order for an app to utilize an APK expansion file, there's quite a number of tricky steps to complete (and/or trip up on), including: 

* Following a strict file naming convention `main.<expansion-version>.<package-name>.obb` 
* Hard-coding the OBB name and version in your app
* Writing code ensuring the OBB file exists locally and is the expected size in bytes once the user downloads the app
* Crafting the code to actually access your files within your extension file
* Uploading the OBB file to the Play Store
* Ensuring the OBB file is set to be used by your newly uploaded store APK
* Adding (a lot) of code to use the 'Google Play App Licensing service' and have the app download the OBB manually from the store if all else fails 🤷‍♂️ 

Ultimately this approach is far from straightforward and jammed full of potential points of failure but thankfully has been _mostly_ documented by Google. 

This leads us nicely to what went wrong...

# What went wrong 🔥

It's December, nearly time for a seasonal vacation and time for us to tie up some loose ends in our project.

A fairly standard looking request falls on my desk.

> We need to update some audio files in the app for the next release. You can do that right?

"Sure", I say, "What could go wrong?" 😅


### Mistake 1 - Assuming MediaPlayer & OBBs play nicely

Changing an OBB file's content is relatively straightforward in theory. You un-zip the existing file, add the new content, update the file's version and some constants in your app, re-zip, and boom you have a new OBB file.

I made the required changes, added some new files, and here's where I made my first mistake.

{% highlight console %}
zip -r main.123.com.example.app.obb audio/
{% endhighlight %}

With that command, I generated the OBB, uploaded it to the Play Store, and downloaded the newly generated build from an internal test track to give it a spin.

_Result: No in-app audio played. Nothing. Not even the pre-existing sounds._

But hang on, I followed all the steps correctly according to the docs?! I remembered to increase the OBB file version, assign it to the APK, and everything in between. Why was my OBB not supplying the audio?

I searched the app's codebase and several hundred lines of long-forgotten audio code until I stumbled upon a `playAudio(fileName)` method with some interesting looking lines

{% highlight java %}
// Some steps removed... 

// Get the OBB file
ZipResourceFile mainExpansionFile = APKExpansionSupport.getAPKExpansionZipFile(
    context,
    BuildConfig.EXPANSION_FILE_VERSION,
    BuildConfig.EXPANSION_FILE_PATCH_VERSION
);

// Find the file within the OBB
AssetFileDescriptor afd = mainExpansionFile.getAssetFileDescriptor(fileName);

// Play the audio
mediaPlayer.setDataSource(afd.getFileDescriptor(), afd.getStartOffset(), afd.getLength());
mediaPlayer.startAsync();
{% endhighlight %}

After staring at this for some time and having a large cup of coffee, it hit me. The `MediaPlayer` object is reading **directly** from the OBB file using `setDataSource`. If the audio is compressed as part of the ZIP process, then surely the `MediaPlayer` cannot read it. Right?

Bingo ✅

In my haste I had failed to RTFM and fallen into such a common trap, Google explicitly mentions within [their documentation](https://developer.android.com/google/play/expansion-files#StorageLocation).

> If you're packaging media files into a ZIP, you can use media playback calls on the files with offset and length controls (such as `MediaPlayer.setDataSource()` and `SoundPool.load()`) without the need to unpack your ZIP. 
> 
> **In order for this to work, you must not perform additional compression on the media files when creating the ZIP packages.** 
> 
> For example, when using the `zip` tool, you should use the `-n` option to specify the file suffixes that should not be compressed:
> 
> `zip -r -n .mp4;.ogg folder/`
>

Sure enough, generating a new OBB file with **no compression** on the audio files worked just fine:

{% highlight console %}
zip -0 -r main.123.com.example.app.obb audio/
{% endhighlight %}

_Result: All in-app audio plays. Even the pre-existing sounds._

🤦‍♂️ D'oh

### Mistake 2 - Assuming an OBB must come from the Play Store

One of the major flaws with the 'APK Expansion file' approach is seemingly the need for it to be served from the Google Play Store.

If you, as a developer, create a build of your app and deploy it to a device/emulator directly from Android Studio or any other source than the Play Store, you will **not** download the OBB file and thus will not be able to access the OBB's content. For me, that meant the in-app audio would not work on test builds being generated by our CI and served through Firebase's App Distribution. The testers would get plenty of builds, but not have access to a true representation of the final app until it was coming from the Play Store.

My second mistake here is also a common one. Assuming the best or _only_ way to test the OBB file is by creating multiple builds on the Play Store's 'Internal Test Track' and having the OBB served to you via the standard means.

In actual fact, you can skip this step and use `adb` to trick your device into thinking it has already downloaded the OBB by pushing the file to a specific location on your file-system. Using `adb shell` and `adb push` we can make the required folder on the device and push the file to it.

The general approach I took here is as follows:

{% highlight console %}
adb shell mkdir -p /sdcard/Android/obb/<package-name>
adb push main.<expansion-version>.<package-name>.obb /sdcard/Android/obb/<package-name>/main.<expansion-version>.<package-name>.obb
{% endhighlight %}

For example:

{% highlight console %}
adb shell mkdir -p /sdcard/Android/obb/com.example.app
adb push main.123.com.example.app.obb /sdcard/Android/obb/com.example.app
{% endhighlight %}

Once this file is successfully pushed to your device, upon opening your app you should be able to access OBB content as if the file had been served by the Play Store ✨

Simply ask your testers to follow similar steps and voilà, the Play Store is no longer a dependency in order to _fully_ test the app!

🤦‍♂️ D'oh

### Mistake 3 - Assuming Android's MediaPlayer is robust

A few days pass, I hear very little from the QA team and I am fully preparing myself for a simple app release and a quiet Christmas.

Then I see one of my unread messages.

> The audio isn't working on Android 9

Excuse me? How so?

> Only the last few seconds of the new audio files are playing

My very first thought here was 'yikes'. This is the sort of bug that most Android developers detest. Platform and potentially device-specific issues that are going to require either some serious research or be a silly little mistake that makes us look incompetent.

The first thing I do is check the new audio files using [ffmpeg](https://ffmpeg.org/) and compare them to the existing files. You can do this fairly easily by using `ffprobe` to print out some key information

{% highlight console %}
ffprobe -v quiet -print_format json -show_format -show_streams audio/my_new_audio_file.m4a
{% endhighlight %}

I compared the output of the `ffprobe` and consulted the ['Supported Media Formats'](https://developer.android.com/guide/topics/media/media-formats) documentation for Android to check everything was in order.

To my surprise, everything seemed normal and acceptable. Both the new and existing files were MPEG-4 with AAC LC encoding, sampled at 48 kHz with the only noticeable difference being the newer files having been recorded in stereo with two audio channels compared to the existing files recorded in mono with a single channel. 

Ok, maybe that's the issue? Maybe `ffmpeg` might save the day here as it's fairly straight forward to convert stereo to mono

{% highlight console %}
ffmpeg -i audio/input.m4a -ac 1 output.m4a
{% endhighlight %}

So I amend the audio files, create a new OBB file, and move it to my device.

_Result: The audio **still** skips._

Bummer. This means the relentless research into 'Android 9 MediaPlayer MPEG 4 audio skip issues' begins.

A number of hours pass and I find **nothing** but a single [unanswered question](https://stackoverflow.com/q/54013967/717778) on StackOverflow and a comment that gets me thinking

> `MediaPlayer` can cause meaningless problems so it is not a good option. You can use ExoPlayer which developed by Google

This is true. The `MediaPlayer` class is a dinosaur when it comes to Android development, it has been around since API version 1 and is a well-known source of bugs across devices. Migrating to the newer, superior `ExoPlayer` is a reasonable solution but unfortunately not something possible with the time constraints that were presented. It also doesn't explain why `MediaPlayer` plays some of my `.m4a` files correctly and skips others _but_ maybe if we re-encode or reformat the audio to something else, it might work a little nicer.

I have a chat with my boss and we decide to try converting all the app's audio to MP3. This is again very easy with `ffmpeg` 

{% highlight console %}
ffmpeg -i audio/input.m4a -codec:v copy -codec:a libmp3lame -q:a 2 output.m4a
{% endhighlight %}

I take a deep breath, generate the OBB and load it onto the device.

_Result: The audio is crystal clear._

![](https://media.giphy.com/media/dIxkmtCuuBQuM9Ux1E/giphy.gif)

🍾🎊

So what _was_ the problem? I hate to disappoint you but I genuinely have no idea. It seems like `MediaPlayer` can be somewhat temperamental in some situations with `MPEG-4` and should be avoided in favour of `ExoPlayer`.

If _you_ think you know, please [get in touch](https://twitter.com/sp4ghetticode). 

🤦‍♂️ D'oh

# Lessons Learned 🎓

There's a lot of lessons to be learned here.

- Do *not* compress your media in OBB files when using `MediaPlayer`
- Do not assume `MediaPlayer` works perfectly for all MPEG-4 files
- If in doubt, MP3 seems best(?) 🤷‍♂️

However in the future:

- Ditch `MediaPlayer` and use `ExoPlayer` where possible
- **Please please ditch the OBB in favour of App Bundles**

In fact, in the second half of 2021, new apps will be *required* to publish with the Android App Bundle on Google Play and no longer support Extention files.

So thankfully, the future just got a little brighter!

### Thanks 🌟

_Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to clap, like, tweet, share, star etc_
