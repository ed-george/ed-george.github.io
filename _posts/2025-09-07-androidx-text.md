---
layout: post
title: "Whats new in... 'AndroidX Text - 1.0.0-alpha01'"
excerpt: "A quick look at one of the latest additions to the AndroidX suite of libraries"
categories: [Android]
comments: true
medium: 
image:
  feature: /post/androidx-text.jpg
  credit: Galen Crout
  creditlink: https://unsplash.com/photos/white-cherry-blossom-hanging-from-a-tree-over-an-asian-style-roof-0_xMuEbpFAQ
---

### Prologue

こんにちは and welcome! 👋

Before we dive into this article, we need to quickly discuss the Japanese language. **Why?** Well, that will all become apparent shortly!

For some context, over the last 12 months, I have slowly been learning Japanese. It's been an incredibly rewarding experience, but one that has been no easy task for someone who _never really_ mastered English (despite being born and raised in the UK 😅).

Unlike English, which uses a single character-based alphabet, Japanese has **three** main writing systems. These all have slightly different purposes, and a simplified overview of this is as follows:

* "Hiragana" which is commonly used for Japanese words and has symbols (known as kana) that represent specific syllables
  * e.g. いぬ - pronounced 'inu', meaning dog 🐶
* "Katakana" which is commonly used for 'loan words', i.e. words borrowed from languages outside of Japanese. It is similar to hiragana in that it shares the syllables it represents but uses different kana.
  * e.g. ホットドッグ - pronounced 'hotto doggu', meaning hot dog 🌭[^1]
* Finally, you also have "Kanji" which, as the odd one out, is derived primarily from Chinese characters. A single kanji symbol can represent an entire word or a concept, while multiple can represent a lot of information all in a very small text footprint. In fact, there are around 50,000 kanji, and even the most literate Japanese speakers know a mere fraction of them! As a novice Japanese student, _these_ are what keep me up at night 🥲
  * e.g. 犬 - pronounced 'inu' which, as you now know, means dog 🐶

Given kanji can often be difficult even for native speakers, in certain settings, you will often find they are written down with their pronunciations in hiragana annotations above each symbol.

>「<ruby>私<rp>(</rp><rt>わたし</rt><rp>)</rp></ruby>はアンドロイド<ruby>開発者<rp>(</rp><rt>かいはつしゃ</rt><rp>)</rp></ruby>です」-  "I am an Android developer"

This concept of showing these annotative pronunciations is known as [Furigana](https://en.wikipedia.org/wiki/Furigana) in Japanese and [ruby characters](https://en.wikipedia.org/wiki/Ruby_character) more generally within typography. Pretty useful right?![^2] We'll come back to this 😌

Moving on, the other interesting (and relevant) fact about Japanese is, like English, it is commonly written and read horizontally **left-to-right** _but_ depending on the context and medium, can be written vertically where it is then read in the opposite **right-to-left** direction. This vertical writing style is called tategaki (<ruby>縦<rt>たて</rt></ruby><ruby>書<rt>が</rt></ruby>き) and is most commonly used in Japanese typography within books, newspapers and manga. However, if you, like me, have ever picked up manga and tried to read it like an English book only to then spoil the surprise ending, you _might_ have already realised this 😂

Let's hold those thoughts, as with that knowledge we can now move onto the main event ✨🙇

## So, what's actually new in AndroidX?

On August 27th 2025, the brand new `androidx.text` library group was [released](https://developer.android.com/jetpack/androidx/releases/text#1.0.0-alpha01) with a single library, `androidx.text:text-vertical:1.0.0-alpha01`, a new dependency providing enhanced support for 'vertical writing capabilities'[^3]. 

As of the time of writing (early Sept 2025), and as with most of the very earliest releases of AndroidX libraries, extremely limited documentation outside of the commits and release notes exists. However, to understand what this library introduces, let's explore the code and the small [sample app](https://android.googlesource.com/platform/frameworks/support/+/b45fe036b64ac1fcbaa71a6532feb1536e3665f0/text/text-vertical) that accompanied the release within the AndroidX codebase.

#### What's in text-vertical:1.0.0-alpha01

One immediate noteworthy quirk with this new library `androidx.text:text-vertical` is that, unlike the majority of AndroidX libraries, it has a minSdk 36 (Android 16) requirement. This does unfortunately mean the library currently only caters for the very latest Android version, but digging a little deeper shows this is due to the introduction of `Paint.VERTICAL_TEXT_FLAG` in Android 16 which is used by the library to ensure the text rendering does indeed perform vertical text layout calculations correctly. So it is worth noting that does likely mean backwards compatibility to meet AndroidX's more general minSdk 23 (Android 6.0) support might not be forthcoming in future. However, from my testing, should the AndroidX developers find a backward compatable alternative for the flag, it _may_ allow for the library to be minSdk 24 (Android 7.0). 

The main entry-point for this library is the `VerticalTextLayout` class which, despite the name, is **not** to be confused with a layout in either the traditional XML/ `View` system sense or the more contemporary compose-based approach either. The class exposes a `Builder` for construction and, once built, requires consumers to call a method to draw the `VerticalTextLayout` result to a `Canvas`.

This is achieved in Jetpack Compose using an approach that is showcased within the sample app:

{% highlight kotlin %}
@Composable
fun VerticalText(text: Spanned, paint: TextPaint, modifier: Modifier = Modifier) {
    var vTextLayout by remember { mutableStateOf<VerticalTextLayout?>(null) }
    Layout(
        modifier =
            modifier.fillMaxSize().drawWithContent {
                drawIntoCanvas { c ->
                    // Draw method used to draw text to Canvas
                    vTextLayout?.draw(c.nativeCanvas, c.nativeCanvas.width.toFloat(), 0f)
                }
            },
        content = {},
    ) { _, constraints ->
        vTextLayout =
            VerticalTextLayout.Builder(
                text = text,
                start = 0,
                end = text.length,
                paint = paint,
                height = constraints.maxHeight.toFloat(),
            )
                .build()
        layout(constraints.maxWidth, constraints.maxHeight) {}
    }
}
{% endhighlight %}

At this time, there is no current official example of how to also do this using a `View` _but_ if that is something you need, you can find my crude attempt at implementing a simiar approach [here](https://gist.github.com/ed-george/4104855f66d9e2ba720006c2ca7eedd1)[^4]. 

#### A Basic Example - Vertical Spans

Within the sample application, a very simple DSL style API is applied to aid the building of the vertical text. It elegantly applies the new library's newly supported spans which can be demonstrated using this basic example using my name and some numbers:

{% highlight kotlin %}
@Composable
fun SpanExampleText(
    paint: TextPaint,
    modifier: Modifier = Modifier
) {
    VerticalText(
        buildVerticalText {
            // Basic spans examples
            // text("エド 123\n")
            // Upright("エド 123\n")
            // Sideways("エド 123\n")
            // TateChuYoko("エド 123\n")
            

            // Example with emphasis
            withStyle(textColor = Color.Red) { text("エド 123\n") }
            withStyle(textColor = Color.Blue) { Upright("エド 123\n") }
            withStyle(textColor = Color.Magenta) { Sideways("エド 123\n") }
            withStyle(textColor = Color.DarkGray) { TateChuYoko("エド 123\n") }
        },
        paint,
        modifier,
    )
}
{% endhighlight %}     

This renders the following output, which shows some subtle differences (and remember we read this top-to-bottom, right-to-left!)

![]({{site.url}}/img/post/content/text1.png)

* The **red** text is the most basic example where the text's paint has the `Paint.VERTICAL_TEXT_FLAG` applied and no other spans. As you can see, the text is displayed vertically *but* numbers are displayed vertically and rotated to display 'sideways' at 90 degrees. You should note that this isn't typically how numbers are normally displayed in this writing style, but we'll discuss that in a second!
* The **blue** text is set to use the `TextOrientationSpan.Upright()` span behind the scenes and displays text in the same orientation as the previous example and now also renders numbers in the same style. As I understand it, there are occasions where this style is used but it is still not primarily how numbers are displayed. 
* The **magenta** text utilises `TextOrientationSpan.Sideways()` as displays all characters rotated 90 degrees to be sideways. The docs note that this "may be useful for orienting text horizontally when the surrounding text is vertical".
* The **grey** text displays text in a common style used to display horizontal text within a vertical flow using `TextOrientationSpan.TextCombineUpright()`. In Japanese, this style is known as <ruby>縦-中<rp>(</rp><rt>たて-ちゅう</rt><rp>)</rp></ruby> <ruby>横<rp>(</rp><rt>よこ</rt><rp>)</rp></ruby> 'tate-chū yoko' which translates to "horizontal in vertical" and is primarily used for legibility using half-width characters. Tate-chū yoko is how numbers are primarily displayed within a vertical span.

#### Final Example - Adding Emphasis

The final example highlights two additional features the library currently supports (as well as my extremely bad Japanese, sorry!)

* 'Bouten' <ruby>傍点<rp>(</rp><rt>ぼうてん</rt><rp>)</rp></ruby>, which are marks or dots placed above text to add emphasis to a phrase, similar to the use of italics in English
* Font shear, which displays text at a 15-degree angle, akin to italics and is also used in situations where emphasis is needed

{% highlight kotlin %}
@Composable
fun FinalExampleText(paint: TextPaint, modifier: Modifier = Modifier) {
    VerticalText(
        buildVerticalText {
            text(
                text = "私はアンドロイド開発者です。",
                rubyMap = mapOf("私" to "わたし", "開発者" to "かいはつしゃ")
            )
            text(text = "\n")
            TateChuYoko(text = "5")
            text(
                text = "歳の犬を飼っています。",
                rubyMap = mapOf("歳" to "さい", "飼" to "か", "犬" to "いぬ")
            )
            text(text = "\n")
            text(
                text = "人間の年齢に換算すると",
                rubyMap = mapOf(
                    "人間" to "にんげん",
                    "年齢" to "ねんれい",
                    "換算" to "かんさん"
                )
            )
            TateChuYoko(text = "36")
            text(text = "歳です。\n", rubyMap = mapOf("歳" to "さい"))
            
            // Add emphasis with bouten
            withEmphasis(style = EmphasisSpan.STYLE_DOT) { text(text = "すごい", rubyMap = mapOf("歳" to "さい")) }
            text("ですね。\n")
            
            // Add emphasis with font shear
            withFontShear { text(text = "読んでくれてありがとう。", rubyMap = mapOf("読" to "よ")) }
        },
        paint,
        modifier,
    )
}
{% endhighlight %}

![]({{site.url}}/img/post/content/text2.png)


## Is this library for me?

Ok, let's be honest, this is currently a niche library heavily tailored to Japanese specifics which may see limited use outside of the handful of use-cases where the specific vertical writing system is needed. So, based on my knowledge of my post's regular audience, it's _probably not_ something you'll be using anytime remotely soon 😅 However, as a developer with a keen interest in equality and user-friendly software, I hope you'd agree this *is* exciting to see Android evolve ever further and start allowing developers to cater for their audiences needs with a consistent expereince, even if those needs are unique geographicaly, culturely or otherwise. 

If you like me found this subject interesting, I highly recommend the Netflix engineering blog post ['Implementing Japanese Subtitles on Netflix'](https://netflixtechblog.com/implementing-japanese-subtitles-on-netflix-c165fbe61989) from 2017. It highlights some of the other quirks of implementing Japanese typography and was a great resource while investigating this topic.

In closing, I'm very excited to see what's next for the `androidx.text` suite of libraries. I highly doubt `androidx.text:word-art` is coming anytime soon, but fingers crossed just in case...[^5]

![]({{site.url}}/img/post/content/word-art.png)

*Thanks as always for reading! I hope you found this post interesting, please feel free to contact me with any feedback at* [*@Sp4ghettiCode*](https://linktr.ee/sp4ghetticode) *and don’t forget to clap, like, share, star etc — It really helps!*

---

### Footnotes

[^1]: After checking online, inu in katakana <ruby>イ<rp>(</rp><rt>い</rt><rp>)</rp></ruby><ruby>ヌ<rp>(</rp><rt>ぬ</rt><rp>)</rp></ruby> _is_ occasionally used in certain specialized contexts such as biology. So, for completeness, you have now learned to write dog in all three writing styles, you are welcome!
[^2]: If this looks useful, you should definitely check out [android-compose-furigana](https://github.com/mainrs/android-compose-furigana) by [@mainrs](https://github.com/mainrs) on GitHub!
[^3]: If you read the prologue, you probably know where this is going 😌
[^4]: I can't remember the last time I wrote a `View`, so please take this with a pinch of salt 😂
[^5]: 📎💬 _It looks like you're trying to write a blog conclusion... would you like some help?_
