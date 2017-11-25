---
layout: post
title: Droidcon London 2017
category:
---

![lily]({{ site.url }}/assets/droidconuk17/lily.jpg){: .center-image }

[DroidconUK](http://uk.droidcon.com/) is a two days conference in London. This edition was on October 26th and 27th. Asaf, a colleague, wanted to attend as well so we flew together on the 25th. Many thanks to [MyHeritage](https://www.myheritage.com) for helping us getting there!

{% twitter https://twitter.com/galex/status/923240150271954944 align=center %}

I was at [DroidconUK](http://uk.droidcon.com/) in 2012 and 2013 and I missed being there ever since! For non-americans [DroidconUK](http://uk.droidcon.com/) is the best Android Conference: you get to attend high level talks **and** meet & talk to well-known developers and Googlers from the Android Community!

## Day 1

#### [Android - A developer's History](https://skillsmatter.com/skillscasts/10011-keynote-android-a-retrospective) by [Chet Haase](https://twitter.com/chethaase) and [Romain Guy](https://twitter.com/romainguy)

This talk was about all the Android versions and devices since the beginning. I started on Android around 2009 when I got convinced by [Christophe Beyls](https://twitter.com/BladeCoder) (who wrote [the Hidden Costs of Kotlin](https://medium.com/@BladeCoder/exploring-kotlins-hidden-costs-part-1-fbb9935d9b62) articles) to try this new platform as I was already a Java developer (on J2EE 🤢). I bought an [HTC Hero](http://cdn.mos.cms.futurecdn.net/93a28ea293d454bd0454e06cdded5885-480-80.jpg) which still boots to this day but is stuck on Android 2.1 and its so outdated it just sits in my office as a piece of nostalgia.

![android history talk]({{ site.url }}/assets/droidconuk17/android-history.jpg){: .center-image }

#### [Data Persistence in Android — There's Room For Improvement](https://skillsmatter.com/skillscasts/10523-data-persistence-in-android-there-s-room-for-improvement) by [Florina Montenescu](https://twitter.com/FMuntenescu)

[Florina](https://twitter.com/FMuntenescu)'s talk was about [Room](https://developer.android.com/topic/libraries/architecture/room.html) and was super interesting. I used Room in [Talking Kotlin](https://play.google.com/store/apps/details?id=il.co.galex.talkingkotlin) and it fits perfectly well with the other [Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html). A nice info I took out of this talk is that when you use **@Query** with **LiveData** the SQL SELECT query runs on a background thread for free:
{% highlight kotlin %}
@Dao
interface ItemDao {
    @Query("SELECT * FROM item order by pubDate DESC")
    fun getAll(): LiveData<List<Item>>
}
{% endhighlight %}
But calling a function annotated with **@Insert**, **@Update** or **@Delete** runs on the **UI Thread**. Two lines are enough to fix that issue thanks to the [Kotlin Coroutines library](https://github.com/Kotlin/kotlinx.coroutines):
{% highlight kotlin %}
async(CommonPool) {
    TKDatabase.getInstance(context)
              .itemDao()
              .insert(items)
}
{% endhighlight %}

{% twitter https://twitter.com/galex/status/923492980119482373 align=center %}

#### [Android Internals](https://skillsmatter.com/skillscasts/10526-android-internals-for-developers) by [Effie Barak](https://twitter.com/CodingChick)

[Effie](https://twitter.com/CodingChick)'s talk (from Pinterest) was about Android Internals. I think every developer should want to know what a Zygote is and why we can't find by reflexion the object behind a system service. Worth watching!

{% twitter https://twitter.com/galex/status/923505947330441217 align=center %}

#### [Vector Workflows](https://skillsmatter.com/skillscasts/10781-reduce-reuse-recycle) by [Nick Butcher](https://twitter.com/crafty)

[Nick Butcher](https://twitter.com/crafty) did a super-dense but mind-blowing talk on Vector Drawables and Animated Vector Drawables. I wanted to switch resources to vector drawables in my apps for a very long time but it seemed so vast and a design job. Nick showed us that it is actually quite simple with the help of [Sketch](https://www.sketchapp.com/) (first time I hear about it) and [shape shifter](http://shapeshifter.design).

{% twitter https://twitter.com/galex/status/923538096406302720 align=center %}

#### [Doo z z z z z e](https://skillsmatter.com/skillscasts/10787-doo-z-z-z-z-z-e) by [Ralf Wondratschek](https://twitter.com/crafty)

[Ralf](https://twitter.com/crafty) (from Evernote) is the author of [android-job](https://github.com/evernote/android-job). Hearing his thoughts on why this library is necessary for background jobs is eye-opening. We actually use this library in [MyHeritage's app](https://play.google.com/store/apps/details?id=air.com.myheritage.mobile&hl=en) and we are quite happy with it.

{% twitter https://twitter.com/galex/status/923550633436876801 align=center %}

#### [Why do we need Clean Architecture](https://skillsmatter.com/skillscasts/11002-why-do-we-need-clean-architecture) by [Igor Wojda](https://twitter.com/igorwojda)

Really interesting talk, worth watching! I've got to admit I feel I fall behind in terms of Clean Architecture and testing best practices. I do implement presenters and write tests but I have a guts feeling I do not do it in a way it gives me a lot of value over my code. Sometimes it does catch a bug but not that often. I will apply much more clean architecture in the near future to write better tests.

{% twitter https://twitter.com/galex/status/923588287054385157 align=center %}

## Day 2

#### [Developers Are Users Too](https://skillsmatter.com/skillscasts/10790-keynote-looking-forward-to-florina-muntenescu-s-keynote-talk) by [Florina Montenescu](https://twitter.com/FMuntenescu)

This talk was mostly inspiring. It's about Design principles applied to create APIs in a user friendly way, because at the end, developers are (API) users too. The talk is being translated to a blogpost series and you can [find the first one here](https://medium.com/google-developers/developers-are-users-too-part-1-c753483a50dc).

{% twitter https://twitter.com/galex/status/923832475218046977 align=center %}

#### [Testing Android apps based on Dagger and RxJava](https://skillsmatter.com/skillscasts/10532-testing-android-apps-based-on-dagger-and-rxjava) by [Fabio Collini](https://twitter.com/fabioCollini)

This talk seemed rich in information but I didn't manage to follow. RxJava and Dagger are both on my learning list for a long time. I tried to use Dagger but I didn't understand how to do that properly, yet. Actually, other Dependency Injection frameworks are popping lately like [Toothpick](https://github.com/stephanenicolas/toothpick) or [Kodein](https://github.com/SalomonBrys/Kodein) so I might give those a chance first.

{% twitter https://twitter.com/galex/status/923848118831108097 align=center %}

#### [Becoming a Master Window Fitter](https://skillsmatter.com/skillscasts/10297-a-talk-by-chris-banes) by [Chris Banes](https://twitter.com/chrisbanes)

The mains subject here was the **Window** on Android and the related subject of **shared element transitions**. I feel like shared element transitions are a new layer on top of our already complicated activities but I should still implement those as it can give a "wow effect" to the users.

{% twitter https://twitter.com/galex/status/923866069055483904 align=center %}

#### [Pragmatic Kotlin on Android](https://skillsmatter.com/skillscasts/10553-pragmatic-kotlin-on-android) by [Josh Skeen](https://twitter.com/mutexkid)

First full talk about [Kotlin](https://kotlinlang.org/), yay! Besides the nice tips from this talk, I was super happy to see that Kotlin talks got the main room (Gate 1) for almost a full day. **The adoption rate of Kotlin is phenomenal within the Android Community!**

{% twitter https://twitter.com/galex/status/923881621761265664 align=center %}

#### [Travelling across Asia - Our journey from Java to Kotlin](https://skillsmatter.com/skillscasts/10533-travelling-across-asia-our-journey-from-java-to-kotlin) by [Maria Neumayer](https://twitter.com/marianeum) and [Amal Kakaiya](https://twitter.com/K4KYA)

[Maria](https://twitter.com/marianeum) and [Amal](https://twitter.com/K4KYA) (both from Deliveroo) shared their story on how they adopted Kotlin in their company. If you were on the fence about Kotlin or if you were considering but not sure how to approach the transition, this talk is totally for you!

{% twitter https://twitter.com/galex/status/923949161074036736 align=center %}

#### [Sinking Your Teeth Into Bytecode](https://skillsmatter.com/skillscasts/10012-keynote-sinking-your-teeth-into-bytecode) by [Jake Wharton](https://twitter.com/JakeWharton)

[Jake](https://twitter.com/JakeWharton) went deep into bytecode till I what I think was Assembly code! Before learning Kotlin, I barely (if not never) thought about looking into the bytecode for the Java code I was writing. With the Kotlin Plugin in IntelliJ, you have an option to look at the bytecode generated by what you write in Kotlin and from there you can decompile this bytecode and see what is the Java equivalent of your Kotlin code. This is super useful and I think this transparent approach helps a lot reassure old Java developers (like me!) that what happens behind the scene is actually quite good.

{% twitter https://twitter.com/galex/status/923954517477134336 align=center %}

## Conclusion

[Here are all the videos of DroidconUK 2017](https://skillsmatter.com/conferences/8265-droidcon-london-2017#skillscasts). Droidcon London is a **Great** conference. I learned so much and it opens the mind to new concepts. I usually go back home full of ideas for new libraries or things to try. The high level of talks is totally worth it. I'll go back next year, for sure!
