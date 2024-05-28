---
layout: post
title: "The elephant in the room: How to update Gradle in your Android project correctly"
excerpt: "I don't want to alarm you, but if you've updated the Gradle version in an Android project, you've probably done it incorrectly at least once"
categories: [Android, Gradle]
comments: true
medium: https://sp4ghetticode.medium.com/the-elephant-in-the-room-how-to-update-gradle-in-your-android-project-correctly-09154fe3d47b
image:
  feature: /post/wrapper.jpg
  credit: James Hammond
  creditlink: https://unsplash.com/photos/elephant-during-daytime-FNVYos3W0AY
---

## A quick introduction 🤗

I don’t want to alarm you, but if you’ve ever updated the Gradle version in your Android project, you’ve *probably* done it incorrectly at least once 🥲

**How do I know this?** Well, first and foremost I was also guilty of this for a long time but since repenting and learning the ‘correct way’, I have seen it occur in many large open-source projects too (and of course, authored [fixes](https://github.com/duckduckgo/Android/pull/4277) to address it).

But don’t just take my word for it, here’s a poll I conducted on X (or ‘Twitter’ for those living under a rock)

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Android Twitter, be honest, how do you go about updating Gradle in your app?<a href="https://twitter.com/hashtag/AndroidDev?src=hash&amp;ref_src=twsrc%5Etfw">#AndroidDev</a></p>&mdash; Ed Holloway-George 🍝 (@Sp4ghettiCode) <a href="https://twitter.com/Sp4ghettiCode/status/1780620090666320022?ref_src=twsrc%5Etfw">April 17, 2024</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

What’s the problem you may ask? In this post, I hope to answer that question and ensure you update Gradle correctly every time going forward!

## How to not update Gradle 🔥

Let’s consider a real-world scenario. You’ve noticed Gradle has [just released](https://gradle.org/releases/) a new version (which at the time of writing is version 8.7). Your app’s project is behind that version, and you want to update it.

Most developers, my past self included, would probably open `gradle-wrapper.properties` and just simply change the `distributionUrl` property as follows:

{% highlight diff %}
diff --git a/src/some-project/gradle/wrapper/gradle-wrapper.properties b/src/some-project/gradle/wrapper/gradle-wrapper.properties
index a3dc3ba8fe3..d4b79aa7c54 100644
--- a/src/some-project/gradle/wrapper/gradle-wrapper.properties
+++ b/src/some-project/gradle/wrapper/gradle-wrapper.properties
@@ -1,6 +1,6 @@
 distributionBase=GRADLE_USER_HOME
 distributionPath=wrapper/dists
-distributionUrl=https\://services.gradle.org/distributions/gradle-8.5-bin.zip
+distributionUrl=https\://services.gradle.org/distributions/gradle-8.7-bin.zip
 networkTimeout=10000
 validateDistributionUrl=true
 zipStoreBase=GRADLE_USER_HOME
{% endhighlight %}

*Alternatively*, you could also use the wrapper task directly by using a command similar to `./gradlew wrapper --gradle-version=8.7` which will effectively just update the `gradle-wrapper.properties` for you. Nice.

Job done right? Well no, not really, and hopefully a handful of observant readers have already spotted the issue.

## How to (properly) update Gradle ✨

To ensure you migrate to the new version correctly, you need to run the `gradlew wrapper` command up to **two times**, depending on the update method you previously chose above.

While editing your `gradle-wrapper.properties`file will ensure your wrapper will correctly pull the new version’s distribution the next time Gradle is run, it will, unfortunately, leave your wrapper files themselves (i.e. the `gradle-wrapper.jar` and `gradlew` scripts) untouched and *out-of-date* 😬 Yikes!

The same applies to calling `wrapper` directly, the first pass will download the new distribution, but won’t touch your `gradle-wrapper.jar` and `gradlew` files unless you run it *a second time*. 🥲

Why is this? Simply, the wrapper can only write the wrapper files for its *own version*, so if you haven’t previously pulled the distribution, it requires another call once it is downloaded to create/update the relevant wrapper files for the new version.

Confused? So was I. However, the [Gradle documentation](https://docs.gradle.org/current/userguide/gradle_wrapper.html#sec:upgrading_wrapper) *does* mention this explicitly:

> If you want all the wrapper files to be completely up-to-date, you will need to run the `wrapper` task a second time.

D’oh.

## Why bother?

Ok, this is something you might have missed, but *why* should we go to the effort of doing this?

Indeed, using an outdated Gradle wrapper won’t stop your project from building, but to ensure you get the very best from Gradle you’d probably want to make sure you’re using the latest and greatest wrapper. This ensures you reap the benefits of doing so, including bug fixes, security updates, performance improvements etc.

For the sake of an extra call to `./gradlew wrapper` it seems pretty worth it to me!

## But wait, there’s more… 😱

If you’ve read my posts before, you’ll know I tend to talk a lot about [mobile security](https://www.spght.dev/archive/) and may have previously seen me present [how to secure Gradle](https://www.youtube.com/watch?v=loIWpxsMNkg) (now also available as a [blog post](https://www.spght.dev/articles/23-07-2023/gradle-security)). Therefore, it would be quite rude of me not to shoe-horn a little sprinkling into this!

By adding a `gradle-distribution-sha256-sum` field to the `./gradlew wrapper`call with a SHA256 checksum, whenever the distribution is downloaded, the checksum of that file will be compared with the string in the supplied property and the build will fail should they differ. As these distribution [checksums are known](https://gradle.org/release-checksums/) for each Gradle version, any unauthorized changes to the Gradle distribution you downloaded will be caught and you will have just saved yourself from a supply-chain attack!

If you wanted to upgrade to Gradle 8.7, you could call the following:

{% highlight shell %}
./gradlew wrapper --gradle-version=8.7 --distribution-type=bin --gradle-distribution-sha256-sum=544c35d6bd849ae8a5ed0bcea39ba677dc40f49df7d1835561582da2009b961d
{% endhighlight %}

🚨 If you do, make sure you run this **twice**. Once to update the distribution and expected SHA-256 and *again* to update the wrapper files themselves

For more on this, make sure you check out my talk!

## TL;DR

If you’re lazy (like me) and skipped to the end, we can summarise the above as follows!

- If you update Gradle via editing your `gradle-wrapper.properties` file, make sure you always run `gradlew wrapper` after
- If you normally update Gradle via the `gradlew wrapper` directly, make sure you run it *twice*! You also get bonus points from your fellow developers for using the `gradle-distribution-sha256-sum` field 💞

Finally, you can also check your wrapper is the expected version by running `./gradlew wrapper --version` 

Of course, if you want to learn *more* about the Gradle wrapper, check out the [documentation](https://docs.gradle.org/current/userguide/gradle_wrapper.html) ✨

## Thanks 🌟

*Thanks as always for reading! I hope you found this post interesting, please feel free to contact me with any feedback at* [*@Sp4ghettiCode*](https://linktr.ee/sp4ghetticode) *and don’t forget to clap, like, share, star etc*
