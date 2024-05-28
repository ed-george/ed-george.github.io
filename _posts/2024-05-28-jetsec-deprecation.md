---
layout: post
title: "Securing the Future: Navigating the Deprecation of Encrypted Shared Preferences"
excerpt: "EncryptedSharedPreferences is now deprecated, but what does this mean for your app and its security?"
categories: [Android, Security, JetSec]
comments: true
medium:
image:
  feature: /post/jetsec-deprecation.jpg
  credit: Annie Spratt
  creditlink: https://unsplash.com/photos/brown-metal-padlock-BwnrxTVn_uU
---

## Background

Sad news everyone. Very recently the Jetpack Security (JetSec) team at Google [quietly deprecated](https://twitter.com/Sp4ghettiCode/status/1786033489675944311) the `androidx.security:security-crypto` dependency (aka “JetSec Crypto”), the library responsible for popular classes such as `EncryptedSharedPreferences`

The knock-on effect is that this move is causing some developers to ask *“What does this deprecation mean for my app and its security?”* and begin frantically looking into alternatives 😬

In this post, we’ll take a look at what this news *really* means for those who have previously embraced `EncryptedSharedPreferences` and also for those who may have been looking to adopt it.



<iframe src="https://cdn.embedly.com/widgets/media.html?type=text%2Fhtml&amp;key=a19fcc184b9711e1b4764040d3dc5c07&amp;schema=twitter&amp;url=https%3A//twitter.com/Sp4ghettiCode/status/1786033489675944311&amp;image=" allowfullscreen="" frameborder="0" height="568" width="680" title="Ed Holloway-George 🍝 on Twitter: &quot;RIP JetSec Crypto 😢 You were a fun little library with big ambitions#AndroidDev pic.twitter.com/9sqt7RT07L / Twitter&quot;" class="fq n ih dk bg" scrolling="no" style="box-sizing: inherit; top: 0px; width: 680px; height: 568px; left: 0px;"></iframe>

F’s in chat for JetSec Crypto 😢

## A recap on JetSec Crypto 🔏

I’ve previously talked about JetSec Crypto [in more detail before](https://www.spght.dev/articles/04-06-2022/owasp-m2), but in case you missed that, let’s quickly re-acquaint ourselves.

The library began life in alpha releases in May 2019, went stable 1.0.0 in April 2021 and has seen infrequent 1.1.0 pre-releases from that date forward before the recent deprecation.

JetSec Crypto consists of only a small handful of classes, primarily wrappers around popular storage mechanisms, (i.e. `EncryptedSharedPreferences` and `EncryptedFile`) that offer convenient methods to encrypt/decrypt the data held within them. The library also provides a wrapper around Google's cryptography library [Tink](https://developers.google.com/tink) through the `MasterKeys` class which, via a straightforward API, allows developers to create secure keysets for the encryption process utilising best practices for Android.

Despite being a relatively infrequently updated and small library, JetSec Crypto has seemingly been widely adopted, thanks namely to the ‘added security’ it provides and the simplistic familiar APIs.

So in light of this news, what are the cases for and against using `EncryptedSharedPreferences` in 2024 post-deprecation?

## Why NOT JetSec Crypto in 2024?

Firstly, why would we not want to use JetSec Crypto post-deprecation?

To some extent, using `EncryptedSharedPreferences` should be seen as a red flag that *perhaps* your app isn’t following security best practices in the first place 🚩

Ask yourself, are you using the library for storing data locally on a device that’s **sensitive**? This would include data such as personally identifiable information, financial data, credentials, etc. If you are, *should you be*? Chances are, **no** **— you shouldn’t**. Any sensitive data should ideally be server-side, only fetched when required, and require some form of authentication to access.

So if `EncryptedSharedPreferences` was your solution to storing sensitive data on the device, you might have bigger problems…

Additionally, now the library is deprecated it’s certainly worth considering that there will (likely) be no further updates. Any [existing bugs](https://issuetracker.google.com/u/1/issues?q=status%3Aopen+componentid%3A618647&s=created_time%3Adesc), performance issues, etc will remain and **not** be patched.

That said, given this library is so small there’s no reason to fret, as you could consider pulling the [key classes out of the current version](https://android.googlesource.com/platform/frameworks/support/+/7905a13172f3d0479aef27e86a7b83d9c4d25640/security/security-crypto) and manually maintaining them within your project directly.
⚠️ However, this approach is not recommended unless you know what you’re doing! Doing this and *just* keeping Tink up to date is a good start, but be aware that any additional changes may put your app at risk if you aren’t already knowledgeable about cryptography and the Android APIs used by JetSec Crpyto. You **have** been warned!

Finally and arguably most importantly, you might just need to ask yourself “What does `EncryptedSharedPreferences` *really* do for me that `SharedPreferences` doesn’t?”

It sounds silly, but given `SharedPreferences` are held locally on the device in the app’s own folder which isn’t accessible by default, is `EncryptedSharedPreferences`really adding that much. Additionally, as of Android 10 (Q), all devices encrypt user data on the filesystem by *default*, adding another level of security.
A common rebuttal to this is “Well, what about rooted users?” and yes, a user with root access *would* be able to read cleartext `SharedPreferences` but once the device’s integrity is compromised is *anything* really safe, `EncryptedSharedPreferences`included? Many experts (and I), would argue that it isn’t. 🥲 Some food for thought there!

## Why JetSec Crypto in 2024?

Ok, so we’ve heard a few reasons why `EncryptedSharedPreferences`might not be the best choice going forward. However, there are undoubtedly still some positives and a handful of reasons to use it!

Firstly, if you have already adopted `EncryptedSharedPreferences` there’s certainly no *immediate* need to move away from it or seek an alternative. The library won’t stop working overnight and you can still manually update the transitive Tink dependency via your project’s Gradle dependency management. If it’s working for you in the short term, great! Just be aware that in the long term, there will be no updates. If that’s an acceptable level of risk for you and your app, then you are all good 😊

Another often-overlooked reason why JetSec Crypto is so popular is the APIs are straightforward and follow the best practices for cryptography in Android. I personally would trust that the JetSec team knew what they were doing and as such, `EncryptedSharedPreferences` is a very attractive option if I ever find myself needing to secure data locally. This won’t change with deprecation and I’d always prefer (and recommend) to use this over rolling my own solution. That’s never a good idea…

Finally, you might also just have no choice and *need* to use `EncryptedSharedPreferences` 🥲 What do I mean? Well, there are a number of industries (FinTech, Banking, Healthcare, etc) that have regulations that all businesses **must** abide by. Depending on your location or the markets your apps are available in, you may well be bound by laws or regulations that dictate any data stored locally **must** be encrypted to some degree. If you are wondering if this applies to you, then you should probably speak to your legal department as I’m not a lawyer, I’m just a lowly developer 😅

## The future 🔮

Unfortunately, I don’t have a crystal ball so I can’t guarantee what comes next. However, it’s been clear from the last handful of major Android releases that security continues to be at the forefront of Google’s mind and providing developers with the tooling and resources to implement secure best practices is not likely something that will disappear any time soon.

I still have my fingers crossed that the JetSec team releases some guidance of their own shortly, but until then we’ll need to wait patiently to see if something completely brand-new comes along and whether we learn for certain whether JetSec is truly dead and buried.

## Conclusion

Ultimately, as the custodian of your own applications, it is *your* decision to make whether `EncryptedSharedPreferences` and JetSec Crypto is the right choice for you and it’s my hope this post will help you make a better-informed decision!

In any case, JetSec Crypto was a great little library that we learned a lot from ✨ I look forward to seeing what comes next in the Android Security world!

As always, I’ll be the first to let you know when we find out more…

## Thanks 🌟

*Thanks as always for reading! I hope you found this post interesting, please feel free to contact me with any feedback at* [*@Sp4ghettiCode*](https://linktr.ee/sp4ghetticode) *and don’t forget to clap, like, share, star etc — It really helps!*
