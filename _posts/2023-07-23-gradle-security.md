---
layout: post
title: "Android Security: Securing your Gradle builds from baddies"
excerpt: "Gradle based-supply chain attacks are sadly nothing new, however there are a number of tools available to avoid them…"
categories: [Android, Security]
comments: true
medium:
image:
  feature: /post/gradle-sec.jpg
  credit: CHUTTERSNAP
  creditlink: https://unsplash.com/photos/fN603qcEA7g
---

This is the accompanying blog post for my recent Droidcon Berlin 2023 talk “How to stop the Gradle Snatchers: Securing your builds from baddies” — you can find the slides, video and other resources for this talk at my site [spght.dev/talks](https://spght.dev/talks) 

---

If you are an Android developer like me, you will likely be somewhat familiar with the ‘elephant in the room’ when it comes to our builds, Gradle. For over 10 years, it has been the go-to build tool for the Android ecosystem replacing ant, and helping developers move forward into a more configurable and pleasant developer experience[^1].

However, like any tool we use in our developer utility belt to build our apps, it is susceptible to security risks that can (and do) pose a threat to our codebases, apps and users.

While my talk and this post approach the topic from an Android development angle, the tips and tricks shared can be applied to **any** Gradle project. In this post, we will explore some simple steps we can take to ensure Gradle doesn’t fall foul to a supply-chain attack. 

#### A supply-what?

A supply-chain attack is loosely defined as an attack on the tooling you use with the intention to introduce vulnerabilities, insecurities or malicious code into the tooling without being detected. It *could* be done internally within an organization by a rouge employee, or perhaps externally by a malicious actor who has gained privileged access to a system or tooling.

These types of attacks, while rare in the mobile industry, *do happen* and Gradle has been proven to be a popular choice of tooling to attack in the past.

Let’s look at three distinct areas where we can make improvements to secure our Gradle projects: **Dependencies**, **Gradle Wrapper** and **Gradle Distribution**.

### Dependencies

It should come as no surprise that most Gradle projects contain external dependencies. Unless you have an extremely large development team, a lot of time or perhaps some incredibly strict security requirements of your own, it’s highly unlikely you will re-write particular code each time you begin a project. Third-party networking libraries, logging or other utilities are common dependencies to add at the start of most Android projects, but once we have added them to our `build.gradle` and we leave it to Gradle to import them from external repositories, **how do we know these dependencies are legitimate** **😧?** 

In 2018, a supply-chain attack was observed via a number of compromised dependencies hosted on the now-defunct Maven repository, JCenter. In Márton Braun’s excellent blog post [‘A Confusing Dependency’](https://web.archive.org/web/20181214053140/http://blog.autsoft.hu/a-confusing-dependency/), he describes how a malicious actor was able to upload artifacts to JCenter that shadowed popular libraries found in other repositories such as Jitpack. These libraries shared package names and common code with their shadowed counterparts but also contained maliciously injected code to run on a user’s device when these libraries were used by unsuspecting developers. From the developer’s standpoint, there would be *little to no obvious difference* when using a legitimate or malicious library, however, the security of their users would be *severely* compromised. The full story itself is best described by Márton, but fortunately JCenter is no-more and we have some tools to protect us from these attacks going forwards.

#### Dependency Verification

When working with dependencies, we have two tools available to *verify* the dependency is legitimate and what we expect. Firstly, we have the ability to check the **integrity** of the dependency (i.e. has the dependency been modified in any way when compared to the *trusted* version) and also the **origin** of the dependency, also known as the provenance (i.e. is the dependency being provided by the legitimate trusted author).

**Integrity** is checked via a checksum verification process. A checksum is a value that can be generated for a given file, via a specific algorithm, that does not change if a file is an exact copy of an original. For example, using the [SHA-256 algorithm](https://gchq.github.io/CyberChef/#recipe=SHA2('256',64,160)&input=SGVsbG8gV29ybGQK), a simple file containing just the string `Hello World` would always resolve to `d2a84f4b8b650…` (You get the idea!). This same concept applies to Gradle dependency artifacts, as if we know the expected checksum, we can verify the integrity of the dependency we download.

To enable these checks via Gradle is as simple as running the following command

{% highlight shell %}
./gradlew --write-verification-metadata sha256 someTaskName
{% endhighlight %}

Running this task[^2] will create a `verification-metadata.xml` file within your `$PROJECT_ROOT/gradle` folder that may look similar to the following:

{% highlight xml %}
<verification-metadata>
   <configuration>
      <verify-metadata>true</verify-metadata>
      <verify-signatures>false</verify-signatures>
   </configuration>
   <components>
      <!-- All your dependencies here! -->
      <component group="com.jakewharton.timber" name="timber" version="5.0.1">
         <artifact name="timber-5.0.1.aar">
            <sha256 value="ffedddfcc8eff42a1604c8577fcfa4b4ffd9f252122c52ea36cfe7967f512f71" />
         </artifact>
      </component>
   </components>
</verification-metadata>
{% endhighlight %}

This metadata file will contain checksums for your dependencies and their artifacts. Any subsequent fetches of these files will be compared with the provided checksum values and should any mismatch occur, a build error will occur — allowing you to catch any suspicious differences quickly. 

In addition to this, an HTML report is also generated within the `$PROJECT_ROOT/build/reports/dependency-verification` folder of your project. This report gives further information surrounding any issues including links to relevant documentation and details on the warnings or errors present.

![](https://i.imgur.com/s4XTd6K.png)

But where are these checksum values actually coming from?

When a dependency is added to a Maven repo, such as Maven Central, the artifacts are uploaded with checksum files with `.md5`, `.sha1`, `.sha256` or `.sha512` file suffixes after the corresponding file’s original  name (e.g `mylib.aar.sha256`). These files contain the hash value for the artifact using the given algorithm and are downloaded from the Maven repo as part of the verification meta-data process. 

![](https://i.imgur.com/rOsvtkw.png)

Once this verification is enabled, when you add or update a dependency you’ll need to ensure *the correct data* for each relevant artifact is added to the metadata file — otherwise your builds will fail and you might get an earful from anyone else working on your project(s)! 😅 If your builds fail without any update to your dependencies, you must investigate as this would be the first indication of a supply-chain attack!

**The provenance**, or origin of a dependency is verified via the signature that was used to sign the artifacts. This signature is generated by a public-private key pair used by the library’s author when uploading the library to a Maven Repo.

To allow for this check to occur in your Gradle builds, amend the previous Gradle task to the following

{% highlight shell %}
./gradlew --write-verification-metadata sha256,pgp someTaskName
{% endhighlight %}

When this is enabled, you’ll see a number of `trusted-key` elements added to your metadata file representing trusted public keys for given dependencies.

{% highlight xml %}
<verification-metadata>
   <configuration>
      <verify-metadata>true</verify-metadata>
      <verify-signatures>true</verify-signatures>
   </configuration>
   <trusted-keys>   
     <trusted-key id=“47bf5922…” group="com.jakewharton.timber" name=“timber" version="5.0.1"/>
   </trusted-keys>
</verification-metadata>
{% endhighlight %}

These trusted public keys are stored locally and tested against the `.asc` PGP signature files that are also present within the Maven repo for the given dependency. When the dependency is fetched from the repo, these files and associated keys are used to authenticate a particular artifact's author. 

⚠️ **PLEASE NOTE:** While the setup of this checksum/signature verification is quite straightforward, for it to provide total security it requires that you begin the process with **complete trust** in the pre-existing dependencies you have imported. Failure to first pre-verify *each* dependency’s checksum against a value you consider trusted renders this process useless. This process also applies to situations where you update the dependency — so while this is a far from frictionless experience, if done properly you can *guarantee* the integrity of your dependencies.

The [Gradle documentation](https://docs.gradle.org/current/userguide/dependency_verification.html) for dependency verification goes into great detail on these two methods. It is an excellent place to find out much more information about how to secure your dependencies and a must-read for any security-conscious developer! 😎

#### Repository Filtering

Another approach to using dependencies in a more secure way is to use [repository filtering](https://docs.gradle.org/current/userguide/declaring_repositories.html#sec:repository-content-filtering). This is the process of creating allow/deny rules for specific repositories surrounding the dependencies they are allowed to fetch. The APIs available for this give developers plenty of scope for creating simple or more complex rules and is something that can be implemented relatively easily through modifications to your existing `repositories` block.

In the example below, we allow *all* dependencies with the group `com.example` and the specific dependency `com.example:foo` to be fetched from JitPack, while excluding all dependencies from `dev.spght.*`[^3]

{% highlight groovy %}
repositories {
    maven {
        url "https://jitpack.io"
        content {
            // Fetch dependencies
            includeGroup "com.example"
            includeModule("com.example", "foo")
            // Exclude dependencies             
            excludeGroupByRegex("dev\\.spght\\..*")
        }
    }
}
{% endhighlight %}

By defining your rules correctly (i.e. in a mutually exclusive way), you can completely guarantee dependencies are downloaded from the source you wish them to be.

However, it should be noted that while this *might* add some degree of trust, this approach **does not verify the dependencies themselves are legitimate** like the checksum or signature checks discussed previously. So certainly consider using repository filtering in combination with other methods. 🙏

#### Gradle Wrapper Verification

The Gradle Wrapper is a tool we use often with little thought. For most of us, the `gradlew` script in our projects is commonly the entry point we use to run specific tasks relating to our projects, such as running tests, building artifacts and everything in between. We might give it no attention, but behind the scenes, the wrapper has an important role in ensuring the “correct version” of Gradle is downloaded and executed for your project.

Part of this process is running the `gradle-wrapper.jar` , a Java executable that is likely checked into your version control. As this file is commonly executed, it is certainly a potential security risk should it be maliciously modified.

In fact, in late 2022 a [supply chain attack on the wrapper](https://blog.gradle.org/wrapper-attack-report) `.jar` file was first observed in the wild within the codebase of an incredibly popular Minecraft Server. A malicious Minecraft plugin, in conjunction with the wrapper JAR executable, was responsible for accessing user data and granting particular users admin/root-level access to the game as well as the underlying tech stack used to serve it. 

On the back of this, Gradle has released a [GitHub Action](https://github.com/marketplace/actions/gradle-wrapper-validation) that can be used as part of your build pipeline to verify that the Wrapper’s JAR is legitimate. It uses the [known wrapper checksums](https://gradle.org/release-checksums/) to ensure the wrapper has not been tampered with and also is smart enough to check files named `gradle-wrapper.jar` with [homoglyph](https://en.wikipedia.org/wiki/Homoglyph) variants (i.e. where an attacker uses a Unicode character that looks similar to an ASCII one in order to deceive developers into using the wrong file). 

However, should you not be using GitHub for CI and require a solution you can run both locally and remotely, I also created a simple [script](https://gist.github.com/ed-george/3751d09ccdfd33cfe48d8987d9f68510) to verify the Wrapper’s JAR

Sadly no official solution exists for verifying the `gradlew` script, but a [GitHub issue](https://github.com/gradle/wrapper-validation-action/issues/16) exists as a feature request. It may be worth keeping an eye on… 👀

#### Gradle Distribution Verification

Finally, it is also possible to verify the actual Gradle distribution that the wrapper downloads. You may have seen the `distributionUrl` property within your `$PROJ_ROOT/gradle/wrapper/gradle-wrapper.properties` file, which contains the URL of the distribution of Gradle to fetch. However, how can we guarantee that what is downloaded is legitimate?

By adding a `distributionSha256Sum` property to this file with a SHA256 checksum, when the distribution is downloaded the checksum of that file will be compared with the string in the supplied property. These distribution [checksums are known](https://gradle.org/release-checksums/) and therefore can be added to your project easily. 

It is also possible to add the `distributionSha256Sum` property dynamically when updating your Gradle wrapper by using the `gradle-distribution-sha256-sum` flag

{% highlight shell %}
./gradlew wrapper --gradle-version=7.5 \
       --gradle-distribution-sha256-sum=cb87f222c…
{% endhighlight %}

Should you download something that doesn’t match the `distributionSha256Sum` checksum, your builds will fail and you can begin to investigate. You never know, this might just save you one day!

### Final Thoughts 💭

Phew, that’s it. Thanks for sticking to the end 😅

In conclusion, while a supply-chain attack on your Gradle tooling might *not* be as common as other forms of attacks, it is still a viable attack vector that has been proven to occur many times over.

The mechanisms Gradle supplies to thwart such attacks are more than simple enough to integrate into any project relatively quickly. Fingers crossed this post helps you make a more informed choice and perhaps look into adding one or more of these to your project.

For more information, check out [spght.dev/talks](https://spght.dev/talks) and catch my talk “How to stop the Gradle Snatchers: Securing your builds from baddies” at Droidcon

Stay safe out there! 💪

### Thanks 🌟

_Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to clap, like, tweet, share, star etc_

#### Further Reading

* [Slides from Droidcon Berlin 23](https://speakerdeck.com/sp4ghetticode/how-to-stop-the-gradle-snatchers)

#### Footnotes

[^1]: If you remember the time before Gradle, you will hopefully be in agreement. In my talk, I refer to this time as ‘the dark ages’.

[^2]: In this example, `someTaskName` is an optional param that, if provided, should be the name of a Gradle task that exposes the dependencies you wish to verify. Gradle’s documentation recommends using the `help` task, but depending on your own configuration, there may be better tasks to run instead.

[^3]: Who could trust someone with a domain that silly?! 😅
