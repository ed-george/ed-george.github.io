---
layout: post
title: "Projecting the Future: Remote IDEs"
excerpt: "JetBrains Projector might just be the tool we need in 2021"
categories: [Android, Tools]
comments: true
medium: https://sp4ghetticode.medium.com/projecting-the-future-remote-ides-beb29814a174
image:
  feature: /post/projector.jpg
  credit: Jeremy Yap
  creditlink: https://unsplash.com/photos/J39X2xX_8CQ
---

It's 2021 and the COVID-19 pandemic is still raging on, causing untold turmoil and meaning "life as we previously knew it" to be a long-forgotten thing of the past. Many of us globally have been forced to reconsider the ways we work, the ways we travel, and even the way we live our daily lives.

However despite this struggle, for developers, there _may_ be a game-changer right around the corner…

# The Background

_Note: This post focuses on Android Studio and mobile development, but this is applicable for most, if not all of IntelliJ's IDE products!_

I am drawing a lot from personal experience here, but I have been remote working as a mobile-developer for close to a year now and I imagine many of you have been in similar-positions too. I was fortunate as my employer provided me with a decent laptop (2019 MBP) meaning I have largely been able to complete my day-to-day role as normal by running Android Studio, writing code and creating builds. 

However, if you have ever used Android Studio you will know that it consumes memory for breakfast. Running an emulator and creating a build will occasionally send my fairly decent MBP into a fan cycle that rivals a 747 take-off. Very rarely, I will lock-up my entire machine due to my nieve assumption that I _might_ be able to multi-task on my \$2500+ laptop.

Ok, snide comments aside, I know I am fairly fortunate that I have a decent machine to work from. But knowing other developers might not be so lucky, and wondering if there is a better option for _all_ out there, I was reminded of the concept of 'remote-build servers'.


# Remote Building 🛠

Remote building mobile-apps is far from a new concept. Many of us will use continuous integration (CI) to generate and distribute builds on a daily basis, however, when I talk about 'remote building' in this context, we must understand we are **not** talking about CI.

What we _are_ talking about is having a dedicated machine(s) that aid developers _during_ the process of writing an application for tasks that we would generally run ourselves locally. i.e. Before an app is ready to be used by others. 

Back in 2020 when anyone could attend meetups without worrying about deadly viruses, I saw a fabulous talk by [Enrique Ramirez](https://twitter.com/kikermo) entitled ['Speeding up your build with remote build servers'
](https://www.youtube.com/watch?v=C__RVKfT5jE). This talk focussed mainly on Enrique's approach to using a remote-machine as a 'build server' to aid his development process when working from home.

The approach used to do this can be broadly summarised as follows:

* Run the IDE and write code locally
* Use `rsync`, `mainfraimer`, `mirikle` or equivalent to sync files between local and a dedicated remote machine
* Compile and build the code on the remote machine
* Sync the results back to the local machine
* Run the build locally as normal

![](https://i.imgur.com/5FrepoF.png)

I implore you to watch Enrique's talk, as it is a fantastic deep-dive into the subject.

The approach he describes means that the 'heavy lifting' is done by the remote machine and the results are synced with the local machine's filesystem. This is obviously _hugely_ advantageous for someone whose local machine is not up to the task of compiling a project but does require some significant time investment in regular maintenance and unfortunately _still_ involves the user having to run a memory-intensive IDE locally.

For most, this might solve some issues and speed up development on a local machine. However, there's also an exciting new tool to consider...

# Enter JetBrains Projector 🎥

On March 11 2021, [JetBrains announced Projector v1.0](https://blog.jetbrains.com/blog/2021/03/11/projector-is-out/) a tool for running JetBrains IDEs (IntelliJ, Android Studio, PyCharm etc) remotely over a network. On the face of it, this sounds like a rather odd tool that most of us would take little notice of. However, in the context of 2021 and the pandemic, I think this is **huge**.

Behind the scenes, Projector uses a custom drawing engine to take the code that would normally render JetBrains' own IDEs on screen and instead passes it through a web-client to be rendered by a recipient browser as a web-page.

This means JetBrains' Projector outright removes the hardware dependency of a local machine to be able to run the IDE. In fact with Projector, _anything_ capable of running a modern browser can now be used to run JetBrains IDEs. A phone, tablet, maybe even your [car](https://twitter.com/Sp4ghettiCode/status/1371114341739720713)? By using Projector's Docker images or setup tools, a remote machine can be used to serve an IDE over a network easily.

This approach does however make the steps for 'remote building' slightly different:

* Run the IDE remotely using Projector
* Access the IDE via your local machine's browser
* Use the browser to use the IDE as if it was local
* Compile and build the code on the remote machine
* Sync the result back to the local machine
* Run the build locally as normal

![](https://i.imgur.com/dmwGZq1.png)

This approach means that the 'heavy lifting' is once again handled by the remote server, as well as the handling of the IDE. The local machine's sole responsibility now is to render the 
IDE in the browser and handle the result of the build (e.g. an Android APK file).

It's not without its faults; This approach currently does not have built-in support for multi-user collaboration, meaning in a team setting this option might not be preferable. But hey, for a v1.0, what do we expect!

Staggeringly, thanks to an [excellent guide](https://github.com/joaquim-verges/ProjectorAndroidStudio/blob/main/README.md) written by [Joaquim Vergès](https://twitter.com/joenrv), this took me less than half-an-hour to set up on my own AWS EC2 instance. 

![](https://i.imgur.com/Kp7ZNc7.png)


# The Future 🔮

You may not have realised it, but I think we have glimpsed at the future of IDEs.

> AWS + JetBrains Projector = Remote Working and $$$?!

With the relatively cheap cost of cloud computing, a world in which remote working is either forced upon us or fast becoming the norm and the knowledge that IDEs don't need to run our own local machines into the ground, I see JetBrains' Projector release to be a really exciting development.

Based on some estimates, this approach to work could cost anywhere between \$50-\$100 per month. Given the cost of my own personal machine, I see this as a seriously viable alternative in a post-COVID world.

It's early days, and I don't know how many companies may adopt this approach but I am certainly going to give this a good trial myself.

Are you interested in trying it out? Let me know in the comments or on Twitter!


### Thanks 🌟

_Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to clap, like, tweet, share, star etc_
