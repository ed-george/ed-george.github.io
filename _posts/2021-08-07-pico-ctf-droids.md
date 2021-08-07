---
layout: post
title: "Learning to 'Hack Android' with picoCTF"
excerpt: "A beginners guide to solving picoCTF's 'droid' challenges"
categories: [Android, Security, Hacking, CTF]
comments: true
medium: https://sp4ghetticode.medium.com/learning-to-hack-android-with-picoctf-4d1334eb6929
image:
  feature: /post/flag.jpg
  credit: Johnny Macri
  creditlink: https://unsplash.com/photos/QQGKsFGBlo8
---

Over the last few weeks and in part thanks to my new fascination with Android app security, I have recently found myself heavily invested in cyber security content online. Youtuber's such as [John Hammond](https://www.youtube.com/channel/UCVeW9qkBjo3zosnqUbG7CFw) and [LiveOverflow](https://www.youtube.com/channel/UClcE-kVhqyiHCcjYwcpfj9w) provide hundreds of hours of free content, teaching the average person (as well as developers such as myself) the core concepts around cyber security, ethical hacking and reverse engineering. 

Part of this content introduced me to cyber security's capture-the-flag ('CTF') events, in which teams or individuals compete on challenges in numerous cyber security disciplines in order to gain points, win prizes and ultimately hold the bragging rights over their peers. The challenges usually range in difficulty and in style, from reverse engineering code to completely taking over a machine and gaining root (admin) access.

However, for the complete novice, [picoCTF](https://picoctf.org/) is a free for all annual event aimed at middle school + high school pupils to introduce them into cyber security through a number of hand crafted challenges. Once the event is over these challenges are released and made accessible for non-event attendees to try them for themselves. Despite the intended audience, the difficulty level of some of the harder challenges makes picoCTF a popular event for many seasoned experts as well as beginners. The aim of each challenge is to find a hidden 'flag' in the format "picoCTF{xxxxxxxxxxx}" using cyber security tools, your own knowledge and very often some expert google-fu.  

What interested me immediately with picoCTF was 5 of their challenges all revolving around Android applications. These [challenges](https://play.picoctf.org/practice?page=1&search=droids), named `droid0` to `droid4` respectively, provided me with an exciting chance to test my Android knowledge and further my understanding of the platform I have worked with for close to 10 years. I thought being an experienced developer would mean they'd all be easy, but how wrong I was!

Below are my solutions to these problems.

### ⚠️ Spoilers Ahead

I highly recommend you try all the [droid challenges](https://play.picoctf.org/practice?page=1&search=droids) before continuing this post to avoid spoilers. They range in difficulty, but with good research technique should be easy enough to solve without too much trouble.

However, should you wish to see the solutions, please continue at your discretion!

#  Challenge 1: droids0

The first challenge provided the following description

> Where do droid logs go. Check out this [file](https://jupiter.challenges.picoctf.org/static/02bcd73e630f50ef0b12bcdad9d84e0d/zero.apk).

The very first step was to download the provided file to check what it was, so using `wget` I downloaded it to my working directory.

{% highlight shell %}
wget https://jupiter.challenges.picoctf.org/static/02bcd73e630f50ef0b12bcdad9d84e0d/zero.apk
{% endhighlight %}

It's an APK file! Great, looks like we are checking out somebody's compiled app.

Assuming it's a debuggable build, it is possible to install this APK on a device or emulator via the [Android Debug Bridge](https://developer.android.com/studio/command-line/adb) 'adb' tool.

{% highlight shell %}
adb install -t zero.apk
{% endhighlight %}

Running the app showed an extremely basic app consisting of a title, input text field, button and another text view.

![](https://i.imgur.com/wQzD5Z0.png)

Playing with the app, inputting text and clicking the button caused the "I'm a flag text" to change to a "Not Today..." message.

With the app's title text and the problem's description, I immediately knew that I needed to check the application's logs.

To do this, I made use of [pidcat](https://github.com/JakeWharton/pidcat) and `grep`'d the output to check for flags.

{% highlight shell %}
pidcat | grep -E -o "picoCTF{.*}"
{% endhighlight %}

Sure enough, there's the flag: <span class="spoilier">picoCTF{a.moose.once.bit.my.sister}</span>

#  Challenge 2: droids1

The second challenge starts with the following description

> Find the pass, get the flag. Check out this [file](https://jupiter.challenges.picoctf.org/static/b12c6d058c7f52eb1fd2015cfd291716/one.apk).

Following the previous challenge, I first downloaded the APK file and installed it to my emulator.

{% highlight shell %}
wget https://jupiter.challenges.picoctf.org/static/b12c6d058c7f52eb1fd2015cfd291716/one.apk

adb install -t one.apk
{% endhighlight %}

The app that installs is a mirror of the previous challenges app, but with the title stating 'brute force not required'. Entering any text and hitting the button provides a helpful 'NOPE'. It looks like a password is first required to gain access to the flag.

Based on the title hint, it suggests that reverse-engineering is the way to go. To reverse engineer the APK we can make use of [apktool](https://ibotpeaches.github.io/Apktool/) to decompile the app, check the resources and even inspect the decompiled Android byte-code written in a [smali format](http://pallergabor.uw.hu/androidblog/dalvik_opcodes.html).

To decompile the app using `apktool` is ultra straightforward. The tool creates a directory `one` with directories for all the decompiled files.

{% highlight shell %}
apktool d one.apk
# Open the relevant Flag file in Sublime Text
subl one/smali/com/hellocmu/picoctf/FlagstaffHill.smali
{% endhighlight %}

Whilst not being a smali expert, opening the smali file I located a `getFlag` method that seemed like a great point to start. Inspecting the method, it was possible to work out what was going on through adding my own annotations.

The annotated file is below:

{% highlight smali %}

// The 'get flag method' presumably provides the flag!
.method public static getFlag(Ljava/lang/String;Landroid/content/Context;)Ljava/lang/String;
    .locals 2
    .param p0, "input"    # Ljava/lang/String;
    .param p1, "ctx"    # Landroid/content/Context;

    .line 11
    const v0, 0x7f0b002f // <-- This looks interesting! Likely a string resource

    // Call context.getString with the value in v0
    invoke-virtual {p1, v0}, Landroid/content/Context;->getString(I)Ljava/lang/String;

    move-result-object v0

    .line 12
    // Create local variable password (presumably from the user input)
    .local v0, "password":Ljava/lang/String;
    // Call String#equals to compare input with the input
    invoke-virtual {p0, v0}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z

    move-result v1

    // Check if the strings are equal
    if-eqz v1, :cond_0

    // If equal, call fenugreek method which returns a String
    // This looks to be a call to a native library that provides the flag
    invoke-static {p0}, Lcom/hellocmu/picoctf/FlagstaffHill;->fenugreek(Ljava/lang/String;)Ljava/lang/String;

{% endhighlight %}

The `0x7f0b002f` in the code referenced a string resource. So to find out which one, I ran a simple `grep`. 

{% highlight shell %}
grep -iFr "0x7f0b002f" .

> ./one/res/values/public.xml:    <public type="string" name="password" id="0x7f0b002f" />
> ./one/smali/com/hellocmu/picoctf/FlagstaffHill.smali:    const v0, 0x7f0b002f
> ./one/smali/com/hellocmu/picoctf/R$string.smali:.field public static final password:I = 0x7f0b002f

{% endhighlight %}

Having found out the relevant string resource is `password`, then we can search the XML files for the relevant password using `find` and `grep`. 

{% highlight shell %}
find . -type f -regex ".*.xml" -exec grep -iFr "password" {} \;

> ./one/res/values/public.xml:    <public type="string" name="password" id="0x7f0b002f" />
> ./one/res/values/strings.xml:    <string name="password">opossum</string>

{% endhighlight %}

Bingo. Entering "opossum" into the app reveals the flag <span class="spoilier">picoCTF{pining.for.the.fjords}</span>

# Challenge 3: droids2

> Find the pass, get the flag. Check out this [file](https://jupiter.challenges.picoctf.org/static/b7d30de6eaaf83e685aea7c10c5bdea8/two.apk).

With the same hint as the last challenge, it was time to follow the same steps as previous.

{% highlight shell %}
wget https://jupiter.challenges.picoctf.org/static/b7d30de6eaaf83e685aea7c10c5bdea8/two.apk

adb install -t two.apk

{% endhighlight %}

Installing the app resulted in the title of the app changing to 'smali sounds like an Ikea bookcase'. In fairness, it totally does. Again, it looked like we'd need to find the password to unlock the flag.

Based on this title, it was a fair assumption we'd be diving head first into the decompiled smali code again.

{% highlight shell %}
apktool d two.apk

# Open the relevant Flag file in Sublime Text
subl one/smali/com/hellocmu/picoctf/FlagstaffHill.smali
{% endhighlight %}

This time the smali code was [far more complex](https://gist.github.com/ed-george/fe5c0e54747bd1514343406c060a7469) than the previous challenge. I decided that to investigate further, I would make use of the popular [jadx decompiler](https://github.com/skylot/jadx) to take the smali and attempt to convert it into something more readable, i.e. Java code.

After downloading `jadx`, I launched the GUI using the `jadx-gui` command and opened `FlagstaffHill.smali`. Thankfully, `jadx` did an excellent job of reversing the smali back into Java.

{% highlight java %}

public class FlagstaffHill
{
    public static String getFlag(final String s, final Context context) {
        final String[] array = { "weatherwax", "ogg", "garlick", "nitt", "aching", "dismass" };
        final int n = 3 - 3;
        final int n2 = 3 / 3 + n;
        final int n3 = n2 + n2 - n;
        final int n4 = 3 + n3;
        if (s.equals("".concat(array[n4]).concat(".").concat(array[n2]).concat(".").concat(array[n]).concat(".").concat(array[n4 + n - n2]).concat(".").concat(array[3]).concat(".").concat(array[n3]))) {
            return sesame(s);
        }
        return "NOPE";
    }

    public static native String sesame(final String p0);
}

{% endhighlight %}

To obfuscate the password, it seems the app was doing some array and string manipulation. Based on a quick look at the code, it looked like the array of words would be placed in a specific order, separated by periods.

However, to avoid the potentially time consuming task of reverse engineering the code and then validating it, I created a new Java file and copied the decompiled code over. From there, I modified the code so it would print out the flag to ensure there was no chance for error and I would get the password immediately.

{% highlight java %}
public class FlagstaffHillModified
{
    public static String getFlag() {
        final String[] array = { "weatherwax", "ogg", "garlick", "nitt", "aching", "dismass" };
        final int n = 3 - 3;
        final int n2 = 3 / 3 + n;
        final int n3 = n2 + n2 - n;
        final int n4 = 3 + n3;
        
        // Return the password
        return "".concat(array[n4]).concat(".").concat(array[n2]).concat(".").concat(array[n]).concat(".").concat(array[n4 + n - n2]).concat(".").concat(array[3]).concat(".").concat(array[n3]);
    }
    
    public static void main(String args[]) {
        // Print the password
        System.out.print("Password: ");
        System.out.println(getFlag());
    }
}

{% endhighlight %}

Once modified, compiling and running the code should provide the expected password.

{% highlight shell %}

javac FlagstaffHillModified.java

java FlagstaffHillModified

> Password: dismass.ogg.weatherwax.aching.nitt.garlick
{% endhighlight %}

Boom. There's the password. With that in hand, I entered it into the app and successfully unlocked the flag <span class="spoilier">picoCTF{what.is.your.favourite.colour}</span>

# Challenge 4: droids3

> Find the pass, get the flag. Check out this [file](https://jupiter.challenges.picoctf.org/static/06318765139795831859f843dd56ce60/three.apk).

Knowing the drill at this point, I followed the same steps as the previous challenges.

{% highlight shell %}
https://jupiter.challenges.picoctf.org/static/06318765139795831859f843dd56ce60/three.apk

adb install -t three.apk

{% endhighlight %}

The hint this time was a little more cryptic

> Make this app your own

Ok? Make it my own? How can I do that.

To see what was going on this time, I again made use of `jadx-gui` and opened the smali file.

{% highlight java %}

public class FlagstaffHill {
    public static native String cilantro(String str);

    public static String nope(String input) {
        return "don't wanna";
    }

    public static String yep(String input) {
        return cilantro(input);
    }

    public static String getFlag(String input, Context ctx) {
        return nope(input);
    }
}

{% endhighlight %}

So it looks like the class only has 4 methods, the `cilantro` native method to get the flag from a compiled library, the `getFlag` method that calls the `nope` method and a seemingly unused `yep` method that gets the flag via `cilantro`. 

Huh. Ok, I think we some how need to modify the app's code to make `getFlag` call `yep` instead of `nope`.

Thankfully, I had just come across the excellent ['Hacktricks'](https://book.hacktricks.xyz) website that documents numerous reverse engineering techniques. The [section on Android](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/smali-changes) pointed out that `apktool` could be used to recompile an app with **modified smali**.

So, I checked the smali for the `getFlag` method

{% highlight smali %}
.method public static getFlag(Ljava/lang/String;Landroid/content/Context;)Ljava/lang/String;
    .locals 1
    .param p0, "input"    # Ljava/lang/String;
    .param p1, "ctx"    # Landroid/content/Context;

    .line 19
    invoke-static {p0}, Lcom/hellocmu/picoctf/FlagstaffHill;->nope(Ljava/lang/String;)Ljava/lang/String;

    move-result-object v0

    .line 20
    .local v0, "flag":Ljava/lang/String;
    return-object v0
.end method
{% endhighlight %}

Could it be as simple as changing `invoke-static {p0}, Lcom/hellocmu/picoctf/FlagstaffHill;->nope(Ljava/lang/String;)Ljava/lang/String;` to use `yep`? I modified the file and followed the steps to recompile the build with my modified code

{% highlight shell %}

# Re-compile the app
# three is the base folder of the decompiled app
apktool b three

# Change directory to the newly generated APK
cd three/dist

# Generate a new key to sign the build
keytool -genkeypair -v -keystore key.keystore -alias publishingdoc -keyalg RSA -keysize 2048 -validity 10000

# Sign the new build
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ./key.keystore three.apk publishingdoc

# Uninstall previous version of the app and install the new one
adb uninstall com.hellocmu.picoctf
adb install three.apk

{% endhighlight %}

Nervously I try clicking the button in the app and to my surprise it reveals the flag <span class="spoilier">picoCTF{tis.but.a.scratch}</span>

# Final Challenge 5: droids4

> Reverse the pass, patch the file, get the flag. Check out this [file](https://jupiter.challenges.picoctf.org/static/926d4bfd7030b13dbc98ca26e608c740/four.apk).

Having done this a few times by this point, I follow the steps from the previous challenges. It seems like that this time it will be a combination of figuring out the password and then modifying smali before recompiling.

{% highlight shell %}
wget https://jupiter.challenges.picoctf.org/static/926d4bfd7030b13dbc98ca26e608c740/four.apk

apktool d four.apk

jadx-gui
{% endhighlight %}

Opening the the smali file in `jadx` shows the following Java code that again makes use of obfuscation techniques to hide the password.

{% highlight java %}

public class FlagstaffHill {
    public static native String cardamom(String str);

    public static String getFlag(String input, Context ctx) {
        StringBuilder ace = new StringBuilder("aaa");
        StringBuilder jack = new StringBuilder("aaa");
        StringBuilder queen = new StringBuilder("aaa");
        StringBuilder king = new StringBuilder("aaa");
        ace.setCharAt(0, (char) (ace.charAt(0) + 4));
        ace.setCharAt(1, (char) (ace.charAt(1) + 19));
        ace.setCharAt(2, (char) (ace.charAt(2) + 18));
        jack.setCharAt(0, (char) (jack.charAt(0) + 7));
        jack.setCharAt(1, (char) (jack.charAt(1) + 0));
        jack.setCharAt(2, (char) (jack.charAt(2) + 1));
        queen.setCharAt(0, (char) (queen.charAt(0) + 0));
        queen.setCharAt(1, (char) (queen.charAt(1) + 11));
        queen.setCharAt(2, (char) (queen.charAt(2) + 15));
        king.setCharAt(0, (char) (king.charAt(0) + 14));
        king.setCharAt(1, (char) (king.charAt(1) + 20));
        king.setCharAt(2, (char) (king.charAt(2) + 15));
        if (input.equals("".concat(queen.toString()).concat(jack.toString()).concat(ace.toString()).concat(king.toString()))) {
            return "call it";
        }
        return "NOPE";
    }
}

{% endhighlight %}

Following the previous challenge. I again made a modified version of the code to print out the password

{% highlight java %}
public class FlagstaffHillModified {

    public static String getFlag() {
        StringBuilder ace = new StringBuilder("aaa");
        StringBuilder jack = new StringBuilder("aaa");
        StringBuilder queen = new StringBuilder("aaa");
        StringBuilder king = new StringBuilder("aaa");
        ace.setCharAt(0, (char) (ace.charAt(0) + 4));
        ace.setCharAt(1, (char) (ace.charAt(1) + 19));
        ace.setCharAt(2, (char) (ace.charAt(2) + 18));
        jack.setCharAt(0, (char) (jack.charAt(0) + 7));
        jack.setCharAt(1, (char) (jack.charAt(1) + 0));
        jack.setCharAt(2, (char) (jack.charAt(2) + 1));
        queen.setCharAt(0, (char) (queen.charAt(0) + 0));
        queen.setCharAt(1, (char) (queen.charAt(1) + 11));
        queen.setCharAt(2, (char) (queen.charAt(2) + 15));
        king.setCharAt(0, (char) (king.charAt(0) + 14));
        king.setCharAt(1, (char) (king.charAt(1) + 20));
        king.setCharAt(2, (char) (king.charAt(2) + 15));
        return "".concat(queen.toString()).concat(jack.toString()).concat(ace.toString()).concat(king.toString());
    }
    
    public static void main(String args[]) {
        System.out.print("Password: ");
        System.out.println(getFlag());
    }

}
{% endhighlight %}

Compiling and running this gave me the password as expected

{% highlight shell %}
javac FlagstaffHillModified.java

java FlagstaffHillModified
> Password: alphabetsoup
{% endhighlight %}

However, having seen the `getFlag` method was hard-coded to always return `"call it"`, I also knew that we'd need to modify the smali code and recompile as per the previous challenge.

I located the smali code that returned the hardcoded string and observed the following

{% highlight smali %}
if-eqz v5, :cond_0

const-string v5, "call it"

return-object v5
{% endhighlight %}

I needed to modify this code to instead invoke the `cardamom` method, and return it.

Thankfully, the previous challenge actually provided the smali code to do this, so it was a simple case of copying it over and replacing the smali for the return method with the following

{% highlight smali %}
invoke-static {p0}, Lcom/hellocmu/picoctf/FlagstaffHill;->cardamom(Ljava/lang/String;)Ljava/lang/String;

move-result-object v0

return-object v0
{% endhighlight %}

With these changes made, it was time to try it out!

{% highlight shell %}

# Re-compile the app
# four is the base folder of the decompiled app
apktool b four

# Change directory to the newly generated APK
cd four/dist

# Generate a new key to sign the build
keytool -genkeypair -v -keystore key.keystore -alias publishingdoc -keyalg RSA -keysize 2048 -validity 10000

# Sign the new build
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore ./key.keystore four.apk publishingdoc

# Uninstall previous version of the app and install the new one
adb uninstall com.hellocmu.picoctf
adb install four.apk

{% endhighlight %}

Entering the previously discovered password and clicking the button reveals the final flag <span class="spoilier">picoCTF{not.particularly.silly}</span>

We've done it! ✨

# Wrapping Up

picoCTF's droid challenges proved to be a fun learning experience with the difficulty starting fairly easy but ramping up and requiring some background research. 

In all these took me a couple of evenings to complete.

I would highly recommend giving this a go yourself as well as looking into the other challenges picoCTF has to offer!

### Thanks 🌟

_Thanks as always for reading! I hope you found this post interesting, please feel free to tweet me with any feedback at [@Sp4ghettiCode](https://twitter.com/sp4ghetticode) and don't forget to clap, like, tweet, share, star etc_
