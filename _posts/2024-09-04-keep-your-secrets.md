---
layout: post
title: "[REDACTED] - How to keep your apps secrets, a secret"
excerpt: "Keeping secrets can be difficult, especially when they are in your app."
categories: [Android, Security]
comments: true
medium:
image:
  feature: /post/jetsec-deprecation.jpg
  credit: Annie Spratt
  creditlink: https://unsplash.com/photos/brown-metal-padlock-BwnrxTVn_uU
---


This blog is a companion to my talk of the same name. Please see the [talks page](https://www.spght.dev/talks) for links to the video and slides.

---

### **[REDACTED]** — How to keep your app’s secrets, a secret!

![img](https://cdn-images-1.medium.com/max/2600/0*xngw5d_lVF0HJ4--)Photo by [Kristina Flour](https://unsplash.com/@tinaflour?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com?utm_source=medium&utm_medium=referral)

### Introduction

Let’s start by stating the obvious, we *all* have secrets 👀 
No, I am not talking about your browser history or what you keep at the back of your sock drawer, I am talking about **secrets in our apps**.

However, hearing that, your mind might immediately jump to the spaghetti code you wrote and have tried desperately to hide from your colleagues, or perhaps the several dozen `//TODO`  comments you authored and then consciously  ignored. Again, luckily for you, this is *not* what this blog aims to discuss and those secrets are safe with you, for now! 🤫

As mobile developers, we often have scenarios that require us to consume or store secrets. The most common form you’re likely familiar with are **API keys** which are usually used by third-party SDKs or services to authenticate and provide additional functionality to your application.

Of course, while having services such as these in your app is *not* inherently a bad thing, misconfiguration or the insecure handling of the API keys by you (the developer) can be hugely problematic. Research by [CyberNews](https://cybernews.com/security/android-apps-leak-hardcoded-secrets) in 2022 assessed that more than 55% of Android applications had secrets including API keys *freely* accessible within the application binaries ¹. The same is true for our iOS friends, with a study finding 68% of the top 100 apps in 2018 suffered from some SDK credential misuse ². Nobody is safe!

Exposure of your app’s secrets in this way could negatively affect your business directly through reputational damage, security breaches, financial impact, decreased stability and many more scary outcomes through attackers using secrets to access services, make malicious requests, steal sensitive data and just generally make a huge mess that will need cleaning up.

Big names in the industry like Uber, Verizon, Accenture, WWE and more have [all been burned by this before](https://www.cloudsek.com/whitepapers-reports/mobile-apps-exposing-aws-keys-affect-100m-users-data), so clearly this is a **real** **issue** 😅 and let’s now look  to fix it 💪

### Step 1: Fix your codebase 📝

Before we even begin to think about any issues in our public app(s), we should first examine the code we write to generate them! 

Keeping secrets present in your codebase is really not a good idea for several reasons:

1. You are likely using version control and therefore also keeping a ‘historical record’ of all your app’s secrets both locally and in your remote repositories. Ask yourself: *“Am I confident the current code and its entire history aren’t exposing sensitive data or keys?”* 
2. If you are using plugins or tooling that accesses/analyzes your source code, you could be leaking your secrets to these services. How certain are you that *they* are keeping your secrets safe?
3. If you’re part of a large development team, you may have dozens or even hundreds of developers that have access to your codebase and history. Can you trust *all* of them to keep your secrets safe? 

I am not trying to scare you (much), but having secrets in your codebase and version control is something we’d very much like to avoid 😅

#### Solution:

To address this, you should consider the following steps:

- **Step 1: Run a tool to detect secrets within your codebase and then continue to run it regularly**

My go-to option for this is usually [gitleaks](https://github.com/gitleaks/gitleaks), a free open-source CLI tool that will check your local git repo for any commonly leaked ‘secrets’ including JWT tokens, AWS keys, Google Cloud API credentials and *much much* more. It’s also configurable, meaning if a particular secret format isn’t present within the defaults it's simple enough to add and see if it's present within your git history.

This is well worth adding to your CI/CD setup, especially if you are able to utilise the [GitHub Action](https://github.com/gitleaks/gitleaks-action) which does much of the setup for you. Making this tool a regular part of your process will ensure you catch any potential issues as quickly as possible.

Even if you're confident you're safe, do yourself a favour and check. You *never* know what might have slipped through!

- **Step 2a: Migrate secrets away from being present in your codebase**

Assuming you've just learned there's a sprinkling of secrets present, you should now look to migrate these to a solution which allows these secrets to not be present in the codebase and therefore, your version control.

One common solution for Android is the Gradle Secrets Plugin. This Gradle plugin, created by Google, allows sensitive secrets to be added to a `secrets.properties` file and injected into an app at compile time to be accessed via a generated `BuildConfig` file or via XML placeholder.

- **Step 2b: Remove historic secrets from your codebase**

Use bfg / git-filter-repo to clean-up git

* **Step 3: Distribute these changes to your team**

Serve the fresh codebase to your hungry developers via a `git push —-force --all`



### Step 2: Secure your secrets 🔐



### Step 3: Get rid of your secrets 🗑️

**Footnotes:**

1.  It was found that Google, Firebase and Facebook were primarily the integrations written by developers that leaked secrets. This fact is even worse when further studies also estimate [roughly 5% of Android apps](https://www.comparitech.com/blog/information-security/firebase-misconfiguration-report/) that utilise Firebase are misconfigured and expose data including personal information, access tokens and passwords if their API key(s) were used nefariously.
2. [Source](https://ieeexplore.ieee.org/abstract/document/8719525/authors#authors) - While my iOS days are somewhat behind me, I highly enjoyed reading the [Secret Management on iOS](https://nshipster.com/secrets/) article on NSHipster. It's worth checking out!
