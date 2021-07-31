---
layout: post
title: "Hands on with Jetpack's Security App Authenticator library"
excerpt: "A look into Jetpack Security's latest exciting addition"
categories: [Android, Jetpack, Security, Tools]
comments: true
medium: https://sp4ghetticode.medium.com/hands-on-with-jetpacks-security-app-authenticator-library-e6e5efd8552b
image:
  feature: /post/app-auth.jpg
  credit: Niv Singer
  creditlink: https://unsplash.com/photos/LkD_IH8_K8k
---

Initially released as an alpha during early May 2021, [Jetpack's App Authenticator library](https://developer.android.com/jetpack/androidx/releases/security#security-app-authenticator_version_100_2) is an exciting addition to the ever growing list of security libraries coming from Android's Jetpack team.

But what is its purpose? In the libraries own words it states the following:

> AppAuthenticator is a new library aimed at simplifying verification of app trust based on signing identity.

In slightly plainer English, `AppAuthenticator` allows developers to query other applications and verify their signing identity against an expected result at runtime. An app's signing identity is the [SHA-256](https://en.wikipedia.org/wiki/SHA-2) of the keystore used to sign the app and whilst the verification of this may on the surface seem a fairly niche requirement for a library to provide, this is incredibly useful for security conscious applications that want to communicate with others through the standard channels of 'Inter Process Communication' (IPC) when such communication occurs between apps that may or may not be signed with the same keys. This is also particularly useful for applications that interact with others through the use of exported services and may wish to pre-verify they are accessing the service of a trusted application through verification of the requested app's identity.

Clearly this is extremely useful in some circumstances, but how can we use the library's code to achieve all this?

# How do we use the library?

**Note: At the time of writing, the library is [currently version alpha-02](https://developer.android.com/jetpack/androidx/releases/security) with minimal provided documentation. It is expected and highly likely that the APIs will change in future releases.**

Firstly, in order to find out your own app's certificate digest for use with this library, Gradle's `signingReport` task can be used. Alternatively, `apksigner` can be used to find the SHA-256 certificate digest for a given `.apk` or `.aab` file.

{% highlight bash %}

# Within your own project
# Will print SHA-256 certificate digest for all app configurations
gradlew signingReport 
# OR with pretty print
gradlew signingReport | grep "SHA-256" | tr '[:upper:]' '[:lower:]' | sed 's/sha-256//g;s/://g'


# With an APK or AAB file
# apksigner is part of the Android SDK build-tools
apksigner verify --print-certs my-apk-file.apk

{% endhighlight %}

The library provides the `AppAuthenticator` class which is initialised through providing a XML config file containing a list of packages and their expected SHA-256 certificate digest that your application can query against. 

Prior to querying an app's identity, the correct permissions must be requested within your `AndroidManifest.xml` file. [As of Android 11](https://medium.com/androiddevelopers/package-visibility-in-android-11-cc857f221cd9), the `<queries>` tag can be used within `<manifest>` to ensure requests to query specific package names or packages that handle specific intents are allowed. This is the preferred approach over the use of the all encapsulating `android.permission.QUERY_ALL_PACKAGES` permission.

Once an app is authorised to query for other packages, the new [app-authentication config file format](https://android.googlesource.com/platform/frameworks/support/+/68207402e3806baade17e482bebef4f7ee4ff3f0/security/security-app-authenticator-testing/src/androidTest/res/raw/test_config.xml) can be used within the standard `res/xml` folder or as an external asset to provide the expected certificate digests for given packages.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<!-- A basic example of the new config file format -->
<app-authenticator>
    <expected-identity>
        <package name="com.example.app">
            <!-- A SHA-256 example -->
            <cert-digest>7d5ac0f764d5ae47a051777bb5fc9a96f30b6b4d3bbb95cddb1c32932fb28b10</cert-digest>
        </package>
    </expected-identity>
</app-authenticator>
{% endhighlight %}

The library exposes two static methods to allow the developer to pass the config file, with either `AppAuthenticator.createFromResource(Context, @XmlRes int)` or `AppAuthenticator.createFromInputStream(Context, InputStream)` generating a valid instance of an `AppAuthenticator` to begin querying the apps locally on the device.

However, as with most Android classes that utilise external configs, the possibility of providing invalid resources or files means these methods should be wrapped with a `try-catch` to ensure no nasty crashes are caused.

{% highlight kotlin %}
try {
    val authenticator = AppAuthenticator.createFromResource(context, R.xml.expected_app_identities)
} catch (ioe: IOException) {
    // Handle IOException
} catch (authException: AppAuthenticatorXmlException) {
    // Handle AppAuthenticatorXmlException
}
{% endhighlight %}

Once a valid instance of an `AppAuthenticator` is available, it is then possible to verify the identity of other apps.

## Verifying identities 👀

![The current available AppAuthenticator API](https://i.imgur.com/is4AMta.png)
The current available `AppAuthenticator` API
{: style="text-align: center;"}

The [available public](https://android.googlesource.com/platform/frameworks/support/+/f0ddb7eefd1d4372430ddbce5a3d723e58999d9b/security/security-app-authenticator/api/current.txt) `AppAuthenticator` API consists of three methods prefixed with `check` and three prefixed with `enforce`. All `check` prefixed calls allow for integrity checks to be made and the result handled by the app by checking against the relevant `AppAuthenticator` result constants. Conversely, the `enforce` methods **ensures** the signature is matched by throwing a `SecurityException` if the package queried does not match the expected signature.

An example of this in practice, once the authenticator instance is configured successfully, would be querying the identity of an app by passing a package name to `checkAppIdentity(String)` and comparing the resulting integer against `SIGNATURE_MATCH` and `SIGNATURE_NO_MATCH`. 

{% highlight kotlin %}
// Using previously defined AppAuthenticator instance
val authenticator = AppAuthenticator.createFromResource(context, R.xml.expected_app_identities)

// Check the identity of a given package name and handle the result
val result = when (authenticator.checkAppIdentity(packageName)) {
    AppAuthenticator.SIGNATURE_MATCH -> "Signature matches"
    AppAuthenticator.SIGNATURE_NO_MATCH -> "Signature does not match"
    else -> return
}

// Enforce the identity of a given package name with error handling
try {
    authenticator.enforceAppIdentity(packageName)
} catch (e: SecurityException) {
    // Handle unexpected signature
}

{% endhighlight %}

The remaining methods can be used to verify the identity of a calling process from an IPC through supplying an expected process id, the expected user id assigned to the application and the name of the permission as defined in the XML config. Thankfully through the use of the standard IPC `Binder` class, these values should be easy enough for pre-existing bound services to access.

# The TL;DR

The new Jetpack Security AppAuthenticator library is a welcomed addition to the growing arsenal of security tools and provides assurance that inter-app communication can be made securely and confidently in future.  

For most developers I imagine this library is unlikely to provide any useful function going forwards, but for those that work with applications that make full use of IPC this is _surely_ a must-have once it moves to a more stable release.

### Thanks 🌟

_Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to clap, like, tweet, share, star etc_
