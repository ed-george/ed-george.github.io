---
layout: post
title: "What's the buzz about the 2024 OWASP Mobile Top 10?"
excerpt: "The OWASP Top 10 list for Mobile has had some significant changes for the first time in nearly a decade.
<br>In this post we'll explore the changes, what this means for your Android apps and how to learn more about the threats we face in 2024."
categories: [Android, Security, OWASP]
comments: true
medium: https://proandroiddev.com/whats-the-buzz-about-the-2024-owasp-mobile-top-10-changes-83c93f765bd3
image:
  feature: /post/owasp-buzz-2024.jpg
  credit: Max Muselmann
  creditlink: https://unsplash.com/photos/brown-insect-5nH0Hh78Nh4
---

### Introduction

First and foremost, a very happy 2024 to you 🎉 I *know* it’s February already, but given this is my first post of the year I hope you can forgive me for being slightly late.

That aside, the good news is this year has already kicked off with some exciting news in the mobile security space. In this post, we’ll discuss the changes that have been made to the OWASP Mobile Top 10 for 2024 and see what this means for **you**, the security-conscious developer!

For those that are already familiar with my OWASP Mobile Top 10 talks or posts, you can totally skip ahead to the more juicy “Top of the OWASP” section and find out what all the excitement is about. However, if you aren’t familiar or would just like a quick refresher, stick around as I recap *what* OWASP is and why the recent changes they made are a **big deal** in mobile security circles.

### What the heck is OWASP? 🐝

If you are new to mobile security, my blog posts, or have just been living under a rock for some time, you might not be aware of OWASP and the excellent work they do.

Founded in 2001, the [Open Worldwide Application Security Project (OWASP) Foundation](https://owasp.org/) is a non-profit that provides documentation, tooling, educational resources and many other community-based services to advance and educate people in software security practises. They are widely acknowledged to be the leading application security community in our field and have many dedicated volunteers within their various projects, including those that touch upon mobile security.

Part of the guidance OWASP provides is their ‘Top 10’ threat lists. These are lists of OWASP’s perceived top 10 threats to the security of particular areas and an incredibly useful resource for any developer [^1]. 

I have previously put out plenty of content surrounding their past guidance, so if you’d like to learn more, please check my [talks page](https://www.spght.dev/talks/) and my previous [OWASP-related blogs](https://www.spght.dev/categories/#OWASP) for far greater details and more relevant links.

If you are a more hands-on learner, there’s also a [companion app](https://github.com/ed-george/owasp-top-five) to my OWASP talks that demonstrates some of the topics outlined.

Regardless, 2024 sees the most significant modifications to the Mobile Top 10 in nearly a decade, so without further ado let’s see what has changed 👀

### “Top of the OWASP” ✨

After a long consultation period and several revisions, the newly released [OWASP Mobile Top Ten 2024](https://owasp.org/www-project-mobile-top-10/) is the third and latest major revision of the Mobile Top Ten list since its initial release in 2014. 

The 2024 release now supersedes the 2016 release and brings some noticeable changes that better reflect the present mobile security landscape, including four brand-new threat categories and a complete re-ordering of threats.

This is all well and good, but what has *actually* changed and what does it all mean? 😅 Well, in true Billboard chart style (or for me as a Brit, [Top of the Pops](https://en.wikipedia.org/wiki/Top_of_the_Pops)), let’s very briefly discuss each threat in reverse order starting at number 10!

![](https://cdn-images-1.medium.com/max/1600/1*9ESds38QHiOyqz_CkOXM7A.gif)[I can hear the theme song…](https://www.youtube.com/watch?v=xWu0OmahYmc)

#### #10: [Insufficient Cryptography](https://owasp.org/www-project-mobile-top-10/2023-risks/m10-insufficient-cryptography.html) ⬇️

Moving down from 2016’s 5th spot, Insufficient Cryptography is the threat posed to mobile apps by poor adoption of modern cryptography best practices. This could be through implementing algorithms that are deemed insecure (SHA-1, MD5, etc), not using secure data transport (HTTPS) or perhaps even by not adding delicious salt to your keys [^2] 🧂.

When utilising cryptography, ensure you follow the best practices for your particular needs such as using algorithms like [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) with *at least* 128-bit block size or [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) with *at least* 2048 bits. If in doubt, defer to a professional or utilise trusted tools such as Google’s [tink](https://developers.google.com/tink) library.

Stay tuned to my blog for a post on this in the future (I hope) [^3].

#### #9: [Insecure Data Storage](https://owasp.org/www-project-mobile-top-10/2023-risks/m9-insecure-data-storage.html) ⬇️

Another big mover from 2016’s list is Insecure Data Storage which moves from the second spot, right down to 9th. Ouch 😣

Your apps may be vulnerable to Insecure Data Storage if they are printing any data (including network calls [^4]) to logs, storing passwords or tokens, creating temporary files or even just using standard storage techniques such as SQL databases or Shared Preferences. 

I’ve covered Insecure Data Storage in detail before, so be sure to [check out my post on it](https://www.spght.dev/articles/04-06-2022/owasp-m2) and ensure your app’s data is stored safely & securely.

As always, it’s worth reiterating that it is a best practice to avoid storing sensitive data on a device whenever possible!

#### #8: [Security Misconfiguration](https://owasp.org/www-project-mobile-top-10/2023-risks/m8-security-misconfiguration.html) ⬆️

On the rise, from the 10th spot is Extraneous Functionality under its new moniker of ‘Security Misconfiguration’. 

If you are aware of the concept of ‘RTFM’ [^5], then you might be able to figure out what this threat is. It’s often caused by developers using incorrect settings in production builds, requesting elevated access or permissions when it’s not required or maybe exposing functionality that was originally intended to be internal to the application.

Documentation can often be your friend. Don’t neglect to read it. 🥲 Grab a coffee and get stuck in!

#### #7: [Insufficient Binary Protections](https://owasp.org/www-project-mobile-top-10/2023-risks/m7-insufficient-binary-protection.html) ⬆️

In a similar vein to famous bands like Cream, Audioslave or Atoms for Peace, by forming a proverbial “supergroup”,  2016’s 8th and 9th spots (Code Tampering & Reverse Engineering) have now combined and moved up the chart to 7th. 😎

Binary Protection focuses on ensuring your app’s binaries (i.e. your `.apk/.aar` for Android or `.ipa` for iOS) do not leak information or can be repackaged. If you are not obfuscating your app correctly, or utilising integrity checks, you may find attackers can reverse engineer or potentially redistribute your apps with malicious code injected.

I highly recommend my good friends at [Guardsquare](https://www.guardsquare.com/) and their excellent tools (e.g. [DexGuard](https://www.guardsquare.com/dexguard) and [Proguard Playground](https://playground.proguard.com/)) to help keep your app safe from these particular threats. Alternatively investing time into learning more about R8 and the Google Play Integrity API may also benefit you!

#### #6: [Inadequate Privacy Controls](https://owasp.org/www-project-mobile-top-10/2023-risks/m6-inadequate-privacy-controls.html) 🆕

A brand new entry at number 6 is ‘Inadequate Privacy Controls’. 

If your app handles a user’s personally identifiable information, i.e. data like full names, precise location, financial details, sexual orientation, etc, which in the wrong hands could be used to impersonate/harass them or possibly commit fraud, then this may apply to you! 🥸

Ensure your app is *never* storing or logging PII locally and you only ever request/transfer the *minimal* amount of information from the user to perform your app’s functionality. This will drastically reduce the chance of PII potentially being exposed through weaknesses in your storage or data transfer. 

For example, should a cinema booking app *require* your sexual orientation or your **precise** location to book tickets? Probably not right! It would be far better to instead request a coarse location to find nearby cinemas and allow the user the option of supplying further personal details to serve recommendations, while also allowing them to opt out at a later date.

#### #5: [Insecure Communication](https://owasp.org/www-project-mobile-top-10/2023-risks/m5-insecure-communication.html) ⬇️

Another mover! 2016’s number 3 Insecure Communication falls two places to 5th. 

These are the threats associated with the transfer or receiving of data. Most applications will be doing that via the internet, but your apps may utilise other communication methods such as NFC or Bluetooth. When there’s data communication, you can bet your last dollar there are risks associated with that!

I won’t go into more detail here, as I have [previously blogged](https://www.spght.dev/articles/12-08-2022/owasp-m3) about Insecure Communication, so be sure to check that out.

#### #4: [Insufficient Input/Output Validation](https://owasp.org/www-project-mobile-top-10/2023-risks/m4-insufficient-input-output-validation.html) 🆕

A brand new entry for number 4 sees us visit the topic of user input and outputs.

It’s incredibly important to ensure both your mobile app and APIs are correctly sanitizing user input when it’s communicated or when it is used as an output 🛁 Failure to do so can potentially lead to threats such as SQL injection or Cross Site Scripting (XSS) which may be used to expose sensitive user data or in more extreme circumstances compromise a device. Make sure you are ensuring you are getting the values and format you expect and discard anything that doesn’t meet that criteria. 

Do you remember the [‘Effective Power’](https://www.youtube.com/watch?v=hJLMSllzoLA) or the [‘Black Spot’](https://www.youtube.com/watch?v=jC4NNUYIIdM)  messages that circulated on WhatsApp? These were issues caused by specifically crafted inputs that when outputted in WhatsApp messages were designed as a ‘Denial of Service’ of sorts. — Could sanitisation have saved the day here? Quite possibly!

In short, by being cautious of I/O, you can sleep easy knowing ‘Little Bobby Tables’ won’t be causing you any issues [^6].

#### #3: [Insecure Authentication/Authorization](https://owasp.org/www-project-mobile-top-10/2023-risks/m3-insecure-authentication-authorization.html) ⬆️

A combo of the previous 4th and 6th places sees insecurities surrounding authentication and authorization make the new 3rd spot.

To avoid any confusion given the similarity in naming, authentication is the act of verifying the user is *who they say they are* while authorization is the act of verifying the user *has the role(s) or credentials* to access a particular resource.

In both cases, it’s recommended that checks for authentication and authorization are always made server-side to avoid vulnerabilities introduced through binary modification or other means. 

If you are communicating with APIs that require authorization, ensure you are using revocable tokens that are tied to the device so tokens can be revoked by users if their device is lost/stolen. Ensure tokens are refreshed periodically and your backend team are correctly authenticating when authorizing access to restricted resources!

Again, for more info on Insecure Authentication, you can [check out my previous blog](https://www.spght.dev/articles/31-08-2023/owasp-m4) on the subject 🤠 Another will follow in due course for Insecure Authorization.

#### #2: [Inadequate Supply Chain Security](https://owasp.org/www-project-mobile-top-10/2023-risks/m2-inadequate-supply-chain-security.html) 🆕

A new entry for the 2nd spot sees the focus shift onto the tools and processes we use to build our apps.

A ‘supply-chain attack’ is an attack on the tooling you use, to introduce vulnerabilities, insecurities or malicious code into the tooling without being detected. It *could* be done internally within your organization by a rouge employee, or perhaps externally by a malicious actor who has gained privileged access to a system or tooling.

To protect against it, a thorough code review process should be part of your regular workflow and regular audits of access control to your supply chain should be performed. You should monitor (or limit) your app’s dependencies and ensure they are also regularly reviewed to avoid introducing vulnerabilities.

If you haven’t checked it out already, I have both previously [blogged](https://www.spght.dev/articles/23-07-2023/gradle-security) and [talked](https://www.droidcon.com/2023/07/31/how-to-stop-the-gradle-snatchers-securing-your-builds-from-baddies/) on the subject of supply chain attacks and how to mitigate them when working with Gradle. 

#### #1: [Improper Credential Usage](https://owasp.org/www-project-mobile-top-10/2023-risks/m1-improper-credential-usage.html) 🆕

Drumroll please 🥁 The biggest threat to mobile security, as determined by OWASP is… **Improper Credential Usage**

This is a catch-all category for cases where there is improper security surrounding credentials, applying to the user’s credentials, API keys and everything in between!

In an alarming 2021 report, an estimated 1 in 200 apps were estimated to be leaking AWS credentials through hardcoded API keys

> Researchers from CloudSek, a cybersecurity vendor based in India, said that a study of some 8,000 apps found that around 0.5%, or one in every 200 mobile apps, contained hardcoded private keys for the APIs that the apps use to communicate with AWS services 
>
> Source: [TechTarget](https://www.techtarget.com/searchsecurity/news/252500361/Popular-mobile-apps-leaking-AWS-keys-exposing-user-data#:~:text=Researchers from CloudSek%2C a cybersecurity,to communicate with AWS services.) & [CloudSek](https://www.cloudsek.com/whitepapers-reports/mobile-apps-exposing-aws-keys-affect-100m-users-data)

An exposed API key or hard-coded credential *could* have devastating outcomes. The leaking of these sorts of credentials has previously caught out major companies such as Uber, Verizon, Accenture and many more, allowing for data breaches where customer data including PII and payment details have been exposed 😅 In general, *any* sensitive data that needs to be stored locally should always be encrypted and the keys used for encryption should be securely stored within a hardware-backed keystore.

This all sounds scary, but avoiding falling foul of improper credential handling yourself is fairly straightforward. Do not store user credentials such as passwords on the device (ever!), ensure you routinely rotate any third-party API keys and ensure you always transmit credentials securely via HTTPS. Simples!

### In Conclusion… 🐝

In conclusion, while there’s been some significant movement between 2016’s and 2024’s list, as a security-conscious developer you should be *always* monitoring your applications for potential threats regardless of their position. The Top 10 might have changed, unfortunately, the reality that mobile security is not overly well understood by many developers has not. By learning more and sharing this knowledge, you can make a difference in changing that!

If there’s to be one takeaway from this post, please make sure you and your teammates are clued up on the potential threats to mobile applications (through great resources like the MASVS or by reading my blogs!) and consider integrating tools such as [AppSweep](http://appsweep.guardsquare.com), [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF), [Snyk](https://snyk.io/) or [SonarQube](https://www.sonarsource.com/products/sonarqube/) into your build pipelines to continuously check your code/binaries for these issues.

Well done for making it to the end! I hope you have a fantastic and secure 2024 🥰 🔐 

### Thanks 🌟

*Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at* [*@Sp4ghettiCode*](https://twitter.com/sp4ghetticode) *and don’t forget to clap, like, tweet, share, star etc*

#### Further Reading

- [owasp-top-five Companion App](https://github.com/ed-george/owasp-top-five) (2016 edition)
- [OWASP Mobile Top Ten 2024](https://owasp.org/www-project-mobile-top-10/)
- Mobile Application Security Verification Standard: [MASVS](https://mas.owasp.org/MASVS/)
- Mobile Application Security Testing Guide: [MASTG](https://mas.owasp.org/MASTG/)

#### Footnotes

[^1]: If you *are* indeed a security-focused mobile developer, you should certainly pay particular attention to the updated [MASVS](https://mas.owasp.org/MASVS/) and [MASTG](https://mas.owasp.org/MASTG/) to apply and test your mobile app’s security model.

[^2]: Nothing to do with sodium chloride. A [salt](https://en.wikipedia.org/wiki/Salt_(cryptography)) in this context is randomised data that should be concatenated to plaintext before being hashed to ensure different hash values are generated, even if the entered plaintext is the same multiple times!

[^3]: Annoyingly, I had the Insufficient Cryptography post mostly written before the 2024 Top 10 was finalized when it was in the 5th spot, not the 10th. (Whose idea was a 10-part blog series anyway!?)

[^4]: *Please* stop logging your network calls in production. It’s 2024. 🥲

[^5]: The ‘F’ is for *fricking* right 😅

[^6]: https://xkcd.com/327/
