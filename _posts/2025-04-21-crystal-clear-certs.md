---
layout: post
title: "How to have 'Crystal Clear Certificates': Securing your Android Apps using Certificate Transparency"
excerpt: "Android 16 introduces official support for Certificate Transparency, but what is it and why is it so important?"
categories: [Android, Security]
comments: true
medium: "https://proandroiddev.com/how-to-have-crystal-clear-certificates-950f1afa2f66"
image:
  feature: /post/certs.jpg
  credit: Ramakant Sharda
  creditlink: https://unsplash.com/photos/three-clear-wine-glasses-P8IJxF4FK30
---

### Background

🎉 Excellent news, everyone! After what feels like a long wait, we are *finally* seeing progress with [Certificate Transparency (CT)](https://certificate.transparency.dev/howctworks/) being made readily available to developers via official APIs as of Android 16.

In this blog post, we’ll discuss what this means, the pros/cons, and of course, how to implement certificate transparency in your applications for Android 16+ and older devices.

#### What even is Certificate Transparency? 📜

Before discussing what changes are available in Android 16, we should take a few minutes to remind ourselves what CT *actually* *is* and why it is important via some useful background info, followed by a quick history lesson. I won’t be detailing all the nuts and bolts or how/why certificates are used, but instead some key details that will help us understand where CT becomes relevant.

Firstly, when talking about certificates, I should probably point out that I am *not* talking about paper certificates you may have received as a kid for swimming 50 meters or being top of the class. No, I am of course talking about the **digital certificates** that are deeply ingrained within modern secure web browsing and are used by our browsers and networking tools to verify the identity of domains. In complete layperson terms, the certificates are part of the process that underpins that little padlock icon in your browser’s address bar 🔐 You know the one right?

Next, we need to discuss entities known as [certificate authorities (CA)](https://www.techtarget.com/searchsecurity/definition/certificate-authority#:~:text=A certificate authority (CA) is,entity with a public key.). A CA forms an important role in this verification process as their responsibility essentially boils down to being a ‘trusted source’ that can perform the verification of both a server’s certificate and determine if secure access to it is possible. Browsers and operating systems usually ship with a list of CAs that they consider trusted sources, and along with that responsibility, there’s a mutual understanding that all CAs follow the most stringent security best practices to ensure their verification process is legitimate and trustworthy. **Trust** is the key word here, and with that in mind, this segues nicely into our brief history lesson.

#### Certificate Transparency History 101 👨‍🏫

In 2011, a Dutch CA called [DigiNotar](https://en.wikipedia.org/wiki/DigiNotar) *was* hacked 😅
This was far from ideal for several reasons:

- At the time, DigiNotar was considered a trusted CA by pretty much *all* major browsers and mobile/desktop operating systems
- Thanks to a pretty big lapse in their own security practices, a breach went unnoticed by DigiNotar for around 5 weeks and led to the attackers issuing around 500 fraudulent certificates for popular sites
- The breach was only brought to DigiNotar’s attention when attackers used these forged certificates to perform a man-in-the-middle (MITM) attack and eavesdrop on Iranian civilians’ Gmail accounts via a fake `*.google.com` certificate
- All affected applications (browsers/OS, etc) then had to scramble and immediately push security updates to remove DigiNotar from their trusted CA lists and prompt their users to update

If you want to learn more about the DigiNotar hack, there’s a great [Darknet Diaries episode](https://open.spotify.com/episode/638vkxv3bwXDaDhmvvay9M?si=4d1ec59093424227) which dives deeper. Give it a listen!

So, in short, DigiNotar swiftly became “DigiNoMore” (they went bust) and the landscape of secure browsing suddenly needed to change to account for any future cases of untrustworthy CAs.

#### Who watches the watchers? 👀

When the DigiNotar hack occurred, the fallout was great enough that it triggered a change in attitude towards certificates.

*How could a fraudulent certificate be issued for a domain by a trusted CA and be spotted quickly?*

The answer, of course, is ✨ certificate transparency ✨

The ‘transparency’ aspect is now achieved by CAs publicly logging when they have issued a new certificate to a number of log servers. Critically, these log servers are ‘append only’ and cryptographically hashed, meaning CAs can only *add* new data to them and cannot be tampered with.

The use of public log servers allows for the auditing of certificate authorities to occur, which in turn means any discrepancies can be caught quickly and dealt with in a timely fashion. Many of the popular browser vendors perform such audits, as well as large tech companies such as Google and Facebook, thus making it significantly harder for any forged or unexpected certificates  to be incorrectly issued through a CA. 

#### Well, what about pinning? 📌

*Ok, so what?* You’ve possibly heard of “certificate pinning” before; it’s supported by Android already, and that’s probably good enough, right?

Well, no. Not quite.

Certificate pinning is different in that only a specific certificate’s public key for a given domain is checked against. This is problematic, particularly in mobile apps, as if the wrong certificate is pinned, or a developer forgets to rotate the pins before a certificate expires, the application will no longer be able to communicate with that endpoint, and an immediate app update would be required. Yikes. 

In this scenario, CT works far better. When a new certificate from a CA is issued, the log server also issues a signed certificate timestamp (SCT) which can then be verified by a client. As there is no longer a reliance on the public keys of certificates, the need to release an app when certificates rotate is eliminated and a new certificate’s authenticity can be checked without disruption.

While this overview glosses over a lot of finer details, Matt Dolan’s talk [“Move over certificate pinning. Certificate Transparency is here!”](https://www.youtube.com/watch?v=p0K_mIxD3a4) gives an **excellent** insight into both certificate pinning and transparency. So don’t just take my word for it!

### Android 16 Changes 🆕

So, with a lot of preamble out of the way, what has *actually* changed as of Android 16 (API 36)?

The [Network Security Config file](https://developer.android.com/privacy-and-security/security-config#FileFormat) now contains a`certificateTransparency` tag allowing CT to be enabled *globally* or *per domain* for Android 16+ devices.

Adding this to your existing security configuration is super straightforward and can be done with just a couple of additional lines.

**Per-Domain:** 

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
  <domain-config>
    <domain includeSubdomains="true">example.com</domain>
      <certificateTransparency enabled="true"/>
  </domain-config>
</network-security-config>
```

**Globally:**

```
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
  <base-config cleartextTrafficPermitted="false">
    <certificateTransparency enabled="true"/>
  </base-config>
</network-security-config>
```

While this is a quick-win, it is worth noting that enabling this feature may interfere with your setup if you are currently allowing user or custom certificates. Given that those certificates may not be public, certificate transparency **will not work**.

You can read the docs for further information [here](https://developer.android.com/privacy-and-security/security-config#certificateTransparency).

#### What about pre-Android 16?

With official Android 16+ support available long term, what support is available to enable CT for pre-Android 16?

The de facto go-to library for this is [Matt Dolan’s Certificate Transparency lib](https://github.com/appmattus/certificatetransparency), which provides support for Java projects as well as Android 4.4 (API 19) and above.

The library provides support for certificate transparency within WebViews via a custom [Java Security Provider](https://docs.oracle.com/javase/8/docs/api/java/security/Provider.html) and also allows for CT with simple integrations for common networking libraries, such as OkHttp, via an HttpInterceptor.

**Basic integration:**

```
val interceptor = certificateTransparencyInterceptor()

val client = OkHttpClient.Builder().apply {
    addNetworkInterceptor(interceptor)
}.build()
```

Importantly, the library also attempts to address the issue of certificate revocation by [providing an interceptor](https://github.com/appmattus/certificatetransparency?tab=readme-ov-file#certificate-revocation) to allow developers to manually define revoked certificates. As Matt points out in his README, you can read more about the problem this attempts to solve [here](https://scotthelme.co.uk/revocation-is-broken//).

### Summary ✨

In conclusion, certificate transparency is certainly an upgrade to certificate pinning and is now officially supported as of Android 16+, bringing Android up-to-date with security best practices *and* iOS, which has had support since around 2016.

If you, like me, are a security-conscious developer, then enabling certificate transparency is a quick win that makes your apps more secure with minimal effort.

So, with all that said, it’s my hope you too will see the clear benefits of transparency 💪

### Thanks 🌟

*Thanks as always for reading! I hope you found this post interesting, please feel free to contact me with any feedback at* [*@Sp4ghettiCode*](https://linktr.ee/sp4ghetticode) *and don’t forget to clap, like, share, star etc — It really helps!*
