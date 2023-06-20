---
layout: post
toc: true
title: "Here's some Android tidbits you may have missed!"
excerpt: "No clickbait, just a collection of bitesize Android goodies I have shared over the years"
categories: [Android]
comments: true
medium:
image:
  feature: /post/tidbits.jpg
  credit: Clem Onojeghuo
  creditlink: https://unsplash.com/photos/6ywyo2qtaZ8
---
{% assign author = site.owner %}

# Introduction

> **tidbit**
>
> *noun* - /ˈtɪd.bɪt/
>
> A small piece of interesting information

Over the years, I have collected a number of very specific bite-sized tidbits that have helped me become a better Android developer, improved my processes or just completely blown my mind. I often share these on my socials, but I commonly forget about them until I inevitably need them once again.

In this post, I aim to continuously document these tips and tricks in the hope having them in one place will help **you**, the reader and me, the incredibly forgetful author. [^1]

If you don't already follow me on <a href="http://twitter.com/{{ author.twitter }}"><i class="fa fa-fw fa-twitter"></i>{{ author.twitter }}</a> or <a href="https://{{ author.mastodon.server }}/@{{ author.mastodon.user }}"><i class="fa-fw fa-brands fa-mastodon"></i>{{ author.mastodon.user }}</a> then please do consider it. There are plenty more of these to come! 

Finally, if you have any tips or tricks of your own to share, can spot a better solution or just want to say 'thanks' **please don't hesitate to do so** via the aforementioned links. I do very much hope that some of these will be useful to you!

# ADB

Here are a number of tidbits I have shared surrounding the Android Debug Bridge tool `adb`

## Easy deeplinking

<img src="{{site.url}}/img/tidbits/tb-adb-deeplink.jpeg"/>

A very common 'gotcha' moment when working with `adb` to deeplink URIs with multiple query parameters is forgetting to ensure the ampersands (&) are correctly escaped. If you fail to do this, you may accidentally only pass the _very first_ query parameter within your URI.

To protect against this, you should either escape the ampersands by using a backslash or wrap the double-quoted string within single quotes.

Here is the helper function I use within my shell's environment file:

{% highlight shell %}
deeplink() {
    echo "Launching deep link: $1"
    adb shell am start -a android.intent.action.VIEW -d "'$1'"
}
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1631627766054395904)

## Disable Chrome

When manually testing code that utilises `androidx.browser` or makes an implicit intent to launch a browser, it can often be very useful to see what happens in the 'extreme case' where a browser app is not available on the device. [^2]

A common way of testing this scenario is by disabling Chrome (and any other browsers) on your device. 

This can also be done via `adb`:
{% highlight shell %}
# Disable Chrome
adb shell pm disable-user --user 0 com.android.chrome

# Enable Chrome
adb shell pm enable com.android.chrome
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1620396205632274434)

## Debug your app on launch

<img src="{{site.url}}/img/tidbits/tb-adb-debug-launch.jpeg"/>

Occasionally you may wish to debug the launch of your application. To do this you can use `adb` to wait for a debugger to attach on your app's launch. This is particularly helpful when launching the app via `adb` for deeplinking.

{% highlight shell %}
# Make app wait for debugger on launch
adb shell am set-debug-app -w "your.app.package"

# Clear wait for debugger
adb shell am clear-debug-app "your.app.package"
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1440319021195280392)

## Disable background processes for app

<img src="{{site.url}}/img/tidbits/tb-background-processes.jpeg"/>

If you need to simulate process death scenarios, you can disable background processes for a given app via `adb` on devices running Android 7 or greater

{% highlight shell %}
# Disable background processes
adb shell cmd appops set "your.app.package" RUN_IN_BACKGROUND ignore

# Enable background processes
adb shell cmd appops set "your.app.package" RUN_IN_BACKGROUND allow
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1493978248941817856)


# Android Studio

Here are a number of tidbits I have shared surrounding our favourite IDE, Android Studio

## Split Java to Kotlin conversion into two commits

<img src="{{site.url}}/img/tidbits/tb-konvert.jpeg"/>

Using Android Studio, it is possible to 'Konvert' [^3] `.java` files to `.kt` files auto-magically using the 'Convert Java File to Kotlin File' functionality. However, by using Android Studio's git view it is possible to split this conversion into two commits. One to rename the file (i.e. `Foo.java` to `Foo.kt`) and one to add the Kotlin code. 

The benefit of this is the original file's history is preserved, instead of the `.java` file being deleted and a new `.kt` file created.

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1494729142436253696)

## Use Live Templates to debug

<img src="{{site.url}}/img/tidbits/tb-live-template.jpeg"/>

Using Android Studio's 'Live Template' feature it is possible to add easy debug logs to a method that also dynamically includes all the parameters.

Use the image above to see how I use it, but the magic used to get a list of parameters is as follows:

{% highlight groovy %}
groovyScript("'' + _1.collect { it + ' = [${' + it + '}]'}.join(', ') + ''", functionParameters())
{% endhighlight %}

You can then call this within methods (here is an example using Timber)

<img src="{{site.url}}/img/tidbits/tb-timber-live-template.gif"/>

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1191683073135525889)

## Autolink Jira Tickets in code comments 

You can auto-magically link to your JIRA tickets (or equivalent) in code comments using the 'Issue Navigation' feature of Android Studio. 

For example, `ABC-1234` here would be a clickable link:
{% highlight kotlin %}
fun myMethod() {
  // Will be fixed in ABC-1234
}
{% endhighlight %}

This can be setup via `Settings | Version Control | Issue Navigation`

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1435982964412854279)


# Kotlin / Android APIs

Here are a number of tidbits I have shared surrounding Kotlin or Android-specific APIs

## Discouraged Annotation 

<img src="{{site.url}}/img/tidbits/tb-discouraged.jpeg"/>

If you don't want to mark something explicitly as `@Deprecated` within your codebase, the `@Discouraged` annotation can be used to highlight that the use of the code is 'not advised'. Lint will also helpfully show a warning within Android Studio for any usages of the annotated code. 

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1610279411902816256)

## Find Files With Kotlin Synthetics

With the announcement that Kotlin Synthetic properties to access views are being completely removed as of Kotlin 1.8, you may want to check how many of your files are affected

If you run the following command at the root of your project, you should get a count of files to migrate

{% highlight shell %}
grep -lr "kotlinx.android.synthetic" --include \*.kt | grep src | wc -l
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1495688599660023812)

## Scope labels in Kotlin

<img src="{{site.url}}/img/tidbits/tb-named-scope.jpeg"/>

A little-known Kotlin language feature is the ability to name lambdas and control scopes using a `name@` label. 

For example, this works really nicely when you have nested scopes and want to explicitly break out of a specific one:

{% highlight kotlin %}
fun foo() {
  run hello@ {
    run world@ {
      return@hello // returns from hello
    }
  }
}
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1400444680777703427)

## Using Plurals Correctly 

<img src="{{site.url}}/img/tidbits/tb-plurals.jpeg"/>

A common face-palm moment is forgetting how to correctly use `Resources#getQuantityString` when we also want to substitute a value into the string.

For example:

{% highlight xml %}
<plurals name="my_plural_string">
  <item quantity="one">%1$d item</item>
  <item quantity="other">%1$d items</item>
</plurals>
{% endhighlight %}

{% highlight kotlin %}
val count: Int = ...

val output = resources.getQuantityString(
  R.plurals.my_plural_string,
  count,
  count // <- Don't forget to add this!
)

// output is either "1 item" or "2 items", "3 items", etc
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1237071302327926784)

# Other

Finally, here's the very best of the rest!

## Add SQLCipher to SQLDelight 

<img src="{{site.url}}/img/tidbits/tb-sqlcipher.jpeg"/>

In my <a href="{{site.url}}/articles/04-06-2022/owasp-m2">"Unpacking Android Security: Part 2 - Insecure Data Storage" blog post</a> I discussed using SQLCipher to encrypt Room databases. 

However, this also works for SQLDelight (or any library where you can pass a custom `SupportSQLiteOpenHelper.Factory` class)

{% highlight kotlin %}
val driver = AndroidSqliteDriver(
    schema = Database.Schema,
    context = applicationContext,
    factory = SupportFactory("<user entered password>".encodeToByteArray())
)
{% endhighlight %}

[Read on Twitter](https://twitter.com/Sp4ghettiCode/status/1538515870024192001)


# Closing Notes

If you have made it this far, I hope you have found a handful of these tips useful. I always enjoy sharing little tricks, so please send me your own.

Wherever possible, I have added the social media links to the original post to give the full context, but if any attribution is missing please let me know and I will update this post ASAP!

Once again, thanks for reading! ✨

# Footnotes

[^1]: Keep this page bookmarked, it may have more goodies the next time you visit!

[^2]: Believe it or not, this _does_ happen, especially when your app is popular enough. I have seen it with my very own eyes. These users and devices exist!

[^3]: Konvert, Kotlinfy, Kotlinize...whatever you want to call it! 
