---
layout: post
title: How to get started with Kotlin
date: 2017-07-10
---
[Kotlin](http://kotlinlang.org/) is a new programming language for the [JVM](https://kotlinlang.org/docs/reference/server-overview.html), [Android](https://kotlinlang.org/docs/reference/android-overview.html), [Javascript](https://kotlinlang.org/docs/reference/js-overview.html), and in the near future, [native (iOS, Windows)](https://github.com/JetBrains/kotlin-native).

I heard about it a long time ago but didn't try it until [Google announced Kotlin became a first class citizen language on Android](https://www.youtube.com/watch?time_continue=6&v=X1RVYt2QKQE). I was genuinely happy to hear this news for two reasons:

1. No more excuses for me to **not** learn it (as an Android Developer)
2. Businesses will be much more inclined to let us, tech people, **use it in production**

So it was decided. I will learn the sh*t of this language!

The question is how? Here's my path, and I hope it will help you to start with Kotlin as well.

## Official documentation

First, just [try it out]((https://try.kotlinlang.org/)). See if you even like it. Do the Koans to check what Kotlin is all about.

Then as it is 100% free, read the [documentation reference](http://kotlinlang.org/docs/reference/) ([also available in PDF](https://kotlinlang.org/docs/kotlin-docs.pdf)). It wasn't that pleasant to read as it is indeed just a reference, not an explanation about why things were defined the way they are. For me, it just doesn't stick into my brain if I don't understand the **why** in addition to the **how**. So to continue and as I love technical books, I ordered the two main ones on Kotlin.

## Books

[Kotlin in Action](https://www.manning.com/books/kotlin-in-action) was written by [Dmitry Jemerov](https://twitter.com/intelliyole?lang=en) and [Svetlana Isakova](https://twitter.com/sveta_isakova?lang=en), two members of the Kotlin dev team, and they explain a lot **why** things are a certain way. The information contained in this book is of high quality, and I advise you to get it and read it, even multiple times!

![Kotlin in Action]({{ site.url }}/assets/kotlin-in-action.jpg){: .center-image }

[Kotlin for Android Developers](https://antonioleiva.com/kotlin-android-developers-book/) ([paper](https://www.amazon.com/gp/product/1530075610/)/[ebook](https://leanpub.com/kotlin-for-android-developers)), written by [Antonio Leiva](https://twitter.com/lime_cl), is an **excellent** book that you can literally read in a few hours. It goes straight to the point with lots and lots of code in it. For Android developers, this book is the right starting point, then only read [Kotlin in Action](https://www.manning.com/books/kotlin-in-action).

![Kotlin For Android Developers]({{ site.url }}/assets/kotlin-for-android-developers.jpg){: .center-image }

## Projects

To learn a new technology, the secret is to use it. Take one of your existing side projects, or create a new one, or take one in production that would be low risk to change and convert it to Kotlin. I am lucky enough to have two Gradle plugins we use at work so I started to rewrite one of them from scratch in Kotlin. I have to say that the experience is super pleasant. It reminds me of the ruby learning days where you just sit in front of your screen in owe as the code is so concise, so clean. And don't even get me started on [coroutines](https://blog.simon-wirtz.de/kotlin-coroutines-guide/)!

## Meetups

Look for a Kotlin [meetup](https://www.meetup.com) group near your place or create one! We as people are all eager to learn, and I am convinced we learn best if we unite and share our knowledge within our communities. I started [KotlinTLV](https://www.meetup.com/KotlinTLV/) and we were 100+ at the first meetup called [Introduction to Kotlin](https://speakerdeck.com/alexgherschon/introduction-to-kotlin).

![Introduction to Kotlin]({{ site.url }}/assets/Introduction-to-kotlin.jpg){: .center-image }

It was **amazing**! We have already 3 more meetups planned as there is so much to talk about with Kotlin: Language features, Platforms, Frameworks, Libraries...

## Next steps

My next step is to start to use Kotlin in our production Android App. It might not be an easy task to convince your team or boss, but as you learned so much with the books, side projects and meetups, I hope you will feel confident that you can be the change engine your team need to go full Kotlin.


I hope this article was helpful to you, and good luck on the Kotlin learning path!
