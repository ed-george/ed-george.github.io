---
layout: post
title: "What's new in Jetpack Security Crypto Version 1.1.0-alpha04"
excerpt: "Out of the blue, the Jetpack Security Crypto library sees its first update in 18 months. Let's take a look at what's new!"
categories: [Android, Security, AndroidX]
comments: true
medium: https://sp4ghetticode.medium.com/whats-new-in-jetpack-security-crypto-version-1-1-0-alpha04-fd850cd12d8a
image:
  feature: /post/sec-alpha04.jpg
  credit: Alexander Schimmeck
  creditlink: https://unsplash.com/photos/Yo1MijFa1KA
---

# Background 

**Note:** If you are not interested in the personal story surrounding this blog and wish to skip to the juicy bit (i.e. [What's New](#whats-new)), please feel free! However, I do think this is too surreal not to tell 😂 

---

Today (9th Nov 2022) was probably _the_ most bizarre day in my relatively short time being a mobile security advocate within the Android community. 

I was fortunate enough to visit this year's [Android Dev Summit](https://developer.android.com/events/dev-summit) in London, which primarily focussed on Google's recent work on 'alternative' form-factors. However, to kick the day off a Modern Android Development (MAD) Q&A panel was fronted by [Nick Butcher](https://twitter.com/crafty) and various other household names in Google's 'Avengers-style' line-up of experts in the field. To the best of my knowledge, this session was not recorded or public-facing but was attended by a packed auditorium of eager developers nonetheless.

Questions from the audience seemed to primarily focus on Baseline Profiles and other topics currently in vogue within the community. However, given this rare opportunity, I wanted an answer to the question that has personally bugged me for nearly 18 months.

Requesting the mic, I composed myself, stepped up and asked

> Is the Jetpack Security library dead?

The panel wasn't sure. I explained how it had been 18 months since the last alpha release and had seemed like a promising library with some interesting features, but its development had seemingly halted without any warning.

*"We'll get back to you."* was the panel's final answer. Totally reasonable. I was just more than grateful to have had the chance to ask and know it was heard by the right people.

However, as the day went on, I was pulled aside to have conversations with multiple in-the-know Googlers (who I shall keep anonymous) about my question. The answers I got varied quite significantly:

* "**No**, it's not dead, is in active development. New features are coming soon."
* "**Yes**, sadly it's dead in the water"
* "**Sorry, we just aren't sure what's going on with it**"

😂 Erm, ok? After these rather contradictory conversations (including a handful of confused tweets/toots from myself), I left having more questions than answers.

Then on my way home from the event, seemingly out of nowhere, the [latest AndroidX](https://developer.android.com/jetpack/androidx/versions/all-channel#november_9_2022) updates are announced and there it was, **1.1.0-alpha04** staring me in the face. After all the day's confusion, a release nobody (not even Googlers) saw coming.

# [What's New](#whats-new)

It's fair to say 540 elapsed days is *not* a typical release cycle for an AndroidX library. However, after nearly 18 months since Jetpack Security Crypto 1.1.0's last alpha version, [the latest alpha04 update](https://developer.android.com/jetpack/androidx/releases/security#1.1.0-alpha04) is long overdue and a very welcome surprise to us all.

### A quick refresher to JetSec

Before we dive into the changes this new alpha brings, let's recap what the JetSec Crypto library brings to the table. You may be forgiven for forgetting during the rather long wait between versions 😊

Version 1.0.0 went stable last April (2021) and brought a handful of useful utilities along with it:

* `EncryptedFile` 
  * To provide encrypted I/O streams to read or write encrypted data to a file.

* `EncryptedSharedPreferences` 
  * To provide a secure implementation of `SharedPreferences` to read or write encrypted key-value-pairs  

* `MasterKeys`
  * An easy way to create secure master encryption keys that can be backed by hardware if required


The 1.0.0 stable release came with out-the-box support for Android Marshmallow 6.0 and above (API 23+) and an associated `ktx` library release to add some syntactic sugar for Kotlin users.

### What's in 1.1.0-alpha04

So, what goodies does this unexpected release actually have?

First and foremost, this release has primarily focussed on fixing a number of long-standing bugs within the library. It isn't jam-packed with new functionality or new features but has plenty of value nonetheless. 

The [release notes](https://developer.android.com/jetpack/androidx/releases/security#1.1.0-alpha04) have 8 points that I will do my best to summarise with some additional context where necessary.

##### API 21 support in ktx 

One of 1.1.0's most noticeable improvements thus far has been the back-porting of the library to support Android 5.0 (API level 21), but frustratingly this was not implemented correctly when using the `ktx` library, meaning it forced a `minSdkVersion` of 23. Finally, 1.1.0-alpha04 has addressed this and [allowed the ktx library to be correctly back-ported to API 21](https://issuetracker.google.com/issues/193550375)! 🙌

However, it should be noted that `MasterKeys` continues to require Android Marshmallow 6.0 (API 23) and this behaviour was updated and cemented in a [number of changes](https://android-review.googlesource.com/#/q/I8b4b8354c197af50300ab37f7d1aeed8fdcd79df) within this release that publicly enforces this through the use of the `RequiresApi` annotation at the class level.

##### Upgrading to Tink 1.7.0

When it comes to encryption, JetSec's heavy lifting is performed by a dependency on Google’s [tink library](https://github.com/google/tink), an open-source project that provides industry-leading crypto algorithms and helps developers follow best practices when performing cryptography. 

It has been regularly updated as part of previous releases but  this time around has jumped from version 1.5.0 in the last JetSec alpha to version 1.7.0 in 1.1.0-alpha04. 

The benefit of this version bump is it also provides additional fixes for issues caused indirectly by tink. The removal of a ['noisy' log message](https://issuetracker.google.com/issues/185219606) in this release is just such an example. It might not be keeping developers awake at night, but is a welcome improvement nonetheless. 

For more on this version upgrade, see [tink's release notes](https://github.com/google/tink/releases/).

##### Minor Bugfixes & Improvements

Of course, with any alpha release, we can expect a sprinkling of very minor bug fixes and improvements. 

For `EncryptedSharedPreferences`, we see an [improvement in error handling](https://issuetracker.google.com/issues/241699427) in cases where the library fails to correctly identify the type of object that was encrypted. A `SecurityException` will now be thrown should the library not identify a supported type of `String`, `Int`, `Long`, `Float`, `Boolean` or `Set<String>`.

Additionally, a fix to the behaviour of `EncryptedSharedPreferences.Editor` was [contributed externally](https://android-review.googlesource.com/c/platform/frameworks/support/+/2163645) (i.e. by a non-Googler) to address the lack of notification to a `OnSharedPreferenceChangeListeners` when the `remove()` method is called. However, it seems this same bug may still exist for the `clear()` method, so fingers  crossed we will see that resolved in alpha05.

`EncryptedFile`s also sees a number of improvements. Firstly, the `openFileInput()` method is [improved](https://android-review.googlesource.com/#/q/I80e415bfd53e9e9f3b9a456d50b6b90c0a00c621) through the more relevant `FileNotFoundException` being thrown in the case of the file not existing. Previously this was a more generic `IOException`.

Finally, a [fix was also added](https://issuetracker.google.com/issues/136590547) for cases where decryption of a `EncryptedFile` was not possible if it was generated in parallel with other encrypted files. In this scenario, a race condition could occur during key generation through `EncryptedFile.Builder` accessing shared prefs under the hood. Any corruption of the key, would make decryption impossible and render your encrypted file useless.

### The TL;DR

In short, this update provides additional stability and sets the groundwork for a continuation of the 1.1.0 release. It remains unclear if we will see any new functionality (`EncryptedDataStore` perhaps?!) but given at least one bug may have slipped through the cracks, let's hope alpha05 or beta01 is right around the corner!

In any case, you can see all the commits in this release [here](https://android.googlesource.com/platform/frameworks/support/+log/66681ad83c328d0dd821b943bb3d375f02c1db61..a1e318590b217ecfce1b2de17eed2f18b6a680bb/security).

### Thanks 🌟

_Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to clap, like, tweet, share, star etc_

**You can now also find me on Mastodon** - [@spaghetticode@androiddev.social](https://androiddev.social/web/@spaghetticode)

#### Further Reading

* [Release Notes](https://developer.android.com/jetpack/androidx/releases/security#1.1.0-alpha04)

