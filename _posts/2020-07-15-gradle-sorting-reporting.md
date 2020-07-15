---
layout: post
title: "How To: Sorting and Reporting Your Dependencies Versions with Gradle"
excerpt: "Working on larger projects can make dependency management difficult, in the post we will look at how to sort and generate a report of our Gradle project's dependencies and their available upgrades"
categories: [Android, Gradle, Tools]
comments: true
medium: https://medium.com/@spaghetticode/how-to-sorting-and-reporting-your-dependencies-versions-with-gradle-6753cfb40bcc
image:
  feature: /post/versioning.jpg
  credit: Andrew Buchanan
  creditlink: https://unsplash.com/photos/GlOlfHtzXVk
---

**Disclaimer:** In a part of this post, we will look at how to generate a report of our Gradle project's dependencies with their available upgrades. My experience with this was in an Android multi-module project but it _should_ apply to just about anything that uses Gradle.

# The Backstory 📚

At work as part of a new development cycle, I was asked to make a note of my app's dependencies, their versions, whether an update to them was available and report these to the dev team and the project owners for review.

> _Wait, hold on. You want this for ALL of the app's dependencies?_ 😬

Given I was working in a multi-module application with only relevant dependencies being used per module, no structured dependency management, and a large number of dependencies, this seemed somewhat challenging and tedious.

My initial thought process to achieve this was along the lines of:

* Find each dependency using `./gradlew :modulename:dependencies`
* Note the current version of each dependency
* For each dependency, check [mvnrepository.com](https://mvnrepository.com) for the latest version
* Create a report with findings to present to the team

Clearly, this would be time-consuming, would scale poorly in the future, and almost certainly have a more elegant solution 🙇‍♂️ **Spoiler: It does.**

The three problems I therefore set out to solve were as follows:

1. Allow for a dependency to have the same version across _all_ my modules
2. Stop re-defining dependencies in multiple modules and try to have a single source of truth
3. Use some sort of "magic" to print out a report to get the out-of-date dependencies

Helpfully, my approach in solving these can be summarised in three simple steps 😌 so let's get down to business!

For the impatient amongst us who want an example app demonstrating my full solution - check the link at the end!


# Step 1: Centralise 🧹
The very first step I took was to **organise my dependencies into one centralised place within the application** with the view of making cross-module versioning easier to manage. 

If you're no stranger to Android development, you'll know there are several different ways of achieving this. The approach I took for this is largely based on something I had seen used before in a sample app by [BracketCove](https://github.com/BracketCove/KotlinMVPCalculator/blob/master/versions.gradle) and within [Android's Architecture Sample](https://github.com/android/architecture-components-samples/blob/master/GithubBrowserSample/versions.gradle). It involves moving the definition of dependencies and their versions into a single `versions.gradle` file with maps of dependencies and versions being generated and made globally accessible to the application's modules.

A small excerpt of this `versions.gradle` approach can be seen here:

{% highlight groovy %}
// In Groovy, [:] refers to an empty map
ext.deps = [:]

// Top level maps
def versions = [:]
def deps = [:]

// ######################################################################################
// # Dependencies
// ######################################################################################

// As an example, here we will define some AndroidX dependencies and some networking dependencies
// But here we would define all versions for plugins, the build and third-party dependencies
// See the sample project for the full implementation of this

// ANDROIDX

def androidx_version = [:]
androidx_version.appcompat = '1.0.2'
androidx_version.ktx_core = '1.0.2'
versions.androidx = androidx_version

def androidx_deps = [:]
androidx_deps.appcompat = "androidx.appcompat:appcompat:$versions.androidx.appcompat"
androidx_deps.ktx_core = "androidx.core:core-ktx:$versions.androidx.ktx_core"
deps.androidx = androidx_deps

// NETWORKING

def network_version = [:]
network_version.retrofit = '2.6.2'
network_version.glide = '4.9.0'
network_version.okhttp = '3.14.2'
versions.network = network_version

def network_deps = [:]
network_deps.retrofit_core = "com.squareup.retrofit2:retrofit:$versions.network.retrofit"
network_deps.retrofit_converter_gson = "com.squareup.retrofit2:converter-gson:$versions.network.retrofit"
network_deps.log_interceptor = "com.squareup.okhttp3:logging-interceptor:$versions.network.okhttp"
network_deps.glide_core = "com.github.bumptech.glide:glide:$versions.network.glide"
network_deps.glide_compiler = "com.github.bumptech.glide:compiler:$versions.network.glide"
deps.network = network_deps

// Create global level dependencies
ext.deps = deps

{% endhighlight %}

As you can see, for each group of dependencies we can define a version number and the full dependency name and group. This has the benefit of simply changing one property to change a dependency version _throughout_ the application and scales nicely as the project grows.

I'd thoroughly recommend checking the full sample file as in my application it is the source of truth for plugin versions, SDK versions and various shared build versions too.

# Step 2: Generalise 👨‍👩‍👧‍👦
Within the `versions.gradle` file we have already grouped similar dependencies in the naming conventions, however, we have yet to add these dependencies into our modules. 

More importantly, we ideally want an elegant way to add these similar dependencies to a module all at once!

The approach I used for this uses another Gradle file `version-groups.gradle` and **groups similar dependencies together across all modules**.

{% highlight groovy %}

// Define a group
def Group(Closure closure) {
    closure.delegate = dependencies
    return closure
}

// Make globally accessible to modules
ext {
    androidx = Group {
        implementation deps.androidx.appcompat
        implementation deps.androidx.ktx_core
    }
    
    network = Group {
        implementation deps.network.retrofit_core
        implementation deps.network.retrofit_converter_gson
        implementation deps.network.log_interceptor
        api deps.network.glide_core
        kapt deps.network.glide_compiler
    }
}
{% endhighlight %}

After grouping these dependencies we can use these in our module-level `build.gradle` files as follows:

{% highlight groovy %}
// Here we define our module's dependencies based on the version-groups we have defined
dependencies {
    // Module specific Dependencies
    kotlin()
    androidx()
    ui()
    testing()
}
{% endhighlight %}

WOW. 😍 Suddenly our dependencies are managed both centrally and cleanly. 

# Step 3: Formalize 💻
Admittedly, this step being called 'formalize' is a bit of a stretch! A more accurate description might be 'Create the report', but that didn't follow the theme 🙃 

Up to now, steps 1 and 2 can be considered 'nice to haves' and totally optional in order to generate a human-readable report, so here comes the meat and potatoes 🍖🥔

After some searching of GitHub, I found the answer to my prayers the [gradle-versions-plugin](https://github.com/ben-manes/gradle-versions-plugin) by [Ben Manes](https://github.com/ben-manes). 

This excellent little Gradle plugin, once integrated, can be used to show your available updates via the terminal using `./gradlew dependencyUpdates`. Using the plugin in that fashion spits out some very useful information such as the example below:

```
The following dependencies have later milestone versions:
 - androidx.appcompat:appcompat [1.0.2 -> 1.3.0-alpha01]
     https://developer.android.com/jetpack/androidx
 - androidx.core:core-ktx [1.0.2 -> 1.5.0-alpha01]
     https://developer.android.com/jetpack/androidx
[...]
```

Great! However, whilst this does give us the answers we are looking for, it isn't exactly easy to read or navigate.

Thankfully, the plugin allows for a custom `outputFormatter` to be defined. This allows us to utilise Groovy's `MarkupBuilder` class to create a webpage using an HTML syntax-style DSL.

To implement this, I created a new file, `dependency-update.gradle` and added the following:

{% highlight groovy %}
import groovy.xml.MarkupBuilder

apply plugin: 'com.github.ben-manes.versions'

def gitSha() {
    return 'git rev-parse --short HEAD'.execute([], rootDir).text.trim()
}

dependencyUpdates {
    outputFormatter = { result ->
        def updatable = result.outdated.dependencies
        if (!updatable.isEmpty()) {
            def filepath = "reports/${project.name}-dependencies-result.html"
            def file = new File(filepath)
            if (!file.exists()) {
                // Make the subdirectories
                file.getParentFile().mkdirs()
            }
            def fileWriter = new FileWriter(file)
            def html = new MarkupBuilder(fileWriter)

            html.html {
                head {
                    // You can define your own style here
                    style("table{width:100%}table,td,th{border:1px solid #4b636e;border-collapse:collapse}td,th{padding:15px;text-align:left}table tr:nth-child(even){background-color:#f5f5f6}table tr:nth-child(odd){background-color:#e1e2e1}table th{background-color:#4b636e;color:#ffffff}")
                }
                body {
                    h3("Module: $project.name")
                    h3("Git commit: ${gitSha()}")
                    h4("Last updated: ${LocalDateTime.now()}")
                    table {
                        tr {
                            th("Group")
                            th("Module")
                            th("Current version")
                            th("Latest version")
                        }
                        updatable.each { dependency ->
                            tr {
                                td(dependency.group)
                                td(dependency.name)
                                td(dependency.version)
                                td(dependency.available.release ?: dependency.available.milestone)
                            }
                        }
                    }
                }
            }
            println "[DEPENDENCY REPORTER] Generated file: $filepath"
        }
    }
}
{% endhighlight %}

We also need to change the top-level `build.gradle` to ensure we apply the plugins, our versioning code, and our newly created dependency report to all modules within our project.

{% highlight groovy %}
buildscript {

    // Get access to the plugins as defined in the versions file
    apply from: "$rootProject.rootDir/versions.gradle"

    // Magic - see the full versions.gradle example
    addRepos(repositories)

    // Define your plugins here - see the full versions.gradle example
    dependencies {
        classpath plugin.kotlin
        classpath plugin.gradle
        classpath plugin.gradle_update_version
    }
}

allprojects {
    addRepos(repositories)
    repositories {
        // Ensure all modules have access to the version groups
        dependencies { apply from: "$rootProject.rootDir/version-groups.gradle" }
    }
}

subprojects {
    // Ensure all modules can report their dependencies
    apply from: "$rootProject.rootDir/dependency-update.gradle"
}
{% endhighlight %}

With all these changes made, what we now get from `./gradlew dependencyUpdates` is an HTML report for each module in our app 🙌 Job done.

![](https://i.imgur.com/Ioqq2Ov.png)



### TL;DR 😴

Here is a link to a sample [GitHub project](https://github.com/ed-george/app-dependency-report-example) that contains a full example of how to configure your Gradle project to get your dependency sorting and reporting 'on fleek'.

### Thanks 🌟

_Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to clap, like, tweet, share, star etc_
