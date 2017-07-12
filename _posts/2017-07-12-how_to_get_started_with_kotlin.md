---
layout: post
title: How to get started with Kotlin
date: 2017-07-12
---

![Kotlin Logo]({{ site.url }}/assets/kotlin-logo.jpg){: .center-image }


[Kotlin](http://kotlinlang.org/) is a new programming language for the [JVM](https://kotlinlang.org/docs/reference/server-overview.html), [Android](https://kotlinlang.org/docs/reference/android-overview.html), [Javascript](https://kotlinlang.org/docs/reference/js-overview.html), and in the near future, [native (iOS, Windows)](https://github.com/JetBrains/kotlin-native).

I heard about it a long time ago but didn't try it until [Google announced Kotlin became a first class citizen language on Android](https://www.youtube.com/watch?time_continue=6&v=X1RVYt2QKQE). I was genuinely happy to hear this news because now companies will be much more inclined to let us, tech people, **use it in production** as Google backs it up.

So it was decided. I will learn this mysterious new language! The question is how? Here's my path, and I hope it will help you to start with Kotlin as well.

## Official documentation

First, just [try it out]((https://try.kotlinlang.org/)). See if you even like it. Do the [Kotlin Koans](https://try.kotlinlang.org/) to check what Kotlin is all about.

Read the [documentation reference](http://kotlinlang.org/docs/reference/) ([also available in PDF](https://kotlinlang.org/docs/kotlin-docs.pdf)), it is 100% free. It wasn't very pleasant to read as it is indeed just a reference, not an explanation about why things were defined the way they are. For me, it just doesn't stick if I don't understand the **why** in addition to the **how**. So to continue, and as I love technical books, I ordered the two main ones on Kotlin.

## Books

[Kotlin in Action](https://www.manning.com/books/kotlin-in-action) was written by [Dmitry Jemerov](https://twitter.com/intelliyole?lang=en) and [Svetlana Isakova](https://twitter.com/sveta_isakova?lang=en), two members of the Kotlin development team. They extensively explain **why** things are the way the are. The information contained in this book is of high quality, so I advise you to get it and read it, even multiple times!

![Kotlin in Action]({{ site.url }}/assets/kotlin-in-action.jpg){: .center-image }

The second book, [Kotlin for Android Developers](https://antonioleiva.com/kotlin-android-developers-book/) ([paper](https://www.amazon.com/gp/product/1530075610/)/[ebook](https://leanpub.com/kotlin-for-android-developers)), written by [Antonio Leiva](https://twitter.com/lime_cl), is an **excellent** book that you can literally read in a few hours. It goes straight to the point with lots and lots of code in it. I would recommend Android developers to start by reading this book first and then continuing with the in-depth [Kotlin in Action](https://www.manning.com/books/kotlin-in-action) book.

![Kotlin For Android Developers]({{ site.url }}/assets/kotlin-for-android-developers.jpg){: .center-image }

## Projects

To learn a new technology, the secret is to use it. Take one of your existing side projects, create a new one or take one in production that would be low risk to change, and convert it to Kotlin. I am lucky enough to have two Gradle Plugins we use at work so I started to rewrite one of them from scratch in Kotlin. I have to say that the experience was very satisfying. It reminds me of days learning of [Ruby on Rails](http://rubyonrails.org/) where you just sit in front of your screen in owe as the code is so concise and clean, and don't even get me started on what [coroutines](https://blog.simon-wirtz.de/kotlin-coroutines-guide/) do to your code!

## Meetups

Look for a Kotlin [meetup](https://www.meetup.com) group near you or create one! I am convinced we learn best when we unite and share our knowledge within our developer communities. I started [KotlinTLV](http://kotlintlv.co.il) and we were 100+ strong at the first meetup called [Introduction to Kotlin](https://www.meetup.com/KotlinTLV/events/240383900/).

![Introduction to Kotlin]({{ site.url }}/assets/introduction-to-kotlin.jpg){: .center-image }

It was **amazing**! We have already 3 more meetups planned as there is so much to talk about with Kotlin: Language features, Platforms, Frameworks, Libraries...

## Next steps

My next step is to start using Kotlin in our production Android App. It might not be an easy task to convince your teammates or bosses, but since you learned so much with the books, side projects and meetups, I hope you will feel confident you can lead the change your team needs to go full Kotlin.

I hope this article was helpful to you, and good luck on the Kotlin learning path!

ps: Thanks to Rhaquel Gherschon, Adi Levi and Guy Tsype for proofreading!
