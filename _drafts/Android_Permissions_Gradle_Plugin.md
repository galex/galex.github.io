---
layout: post
type: drafts
title: Android Permissions Gradle Plugin
category: android dev gradle plugin permissions marshmallow
---
My first Gradle Plugin, [Android Permissions Gradle Plugin](https://github.com/galex/android-permissions-gradle-plugin) was just deployed into jCenter. Yay!

Developing this was a great experience. I use gradle heavily as any other developer as the main project I'm working on has lots of flavors. But the need never arose for a Gradle Plugin until recently, when I wanted to compile the SDK with the latest Android version, 23.

What is the issue with the new Android Permissions System in Marshmallow? Nothing, it's a great feature. It was long time due in my opinion. As a developer you have to request on runtime the dangerous permissions you have declared in your Android Manifest. But there is no API (that I'm aware of!) to request from Android the list of your declared Dangerous Permissions.

So this plugin gives you just that: a list of your declared dangerous permissions, statically, in your code and some useful static methods, per variant.

To build a gradle plugin, you will need some documentation:

- [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html): How to develop with gradle components (tasks, etc)
- [Gradle DSL](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html): What methods/variables are available in Gradle for each gradle component
- [Android Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide) How to use the Android gradle plugin to build apps/libraries
- [Android Gradle Plugin DSL](http://google.github.io/android-gradle-dsl/current/) What gradle components the Android plugin adds
- [Android Gradle Plugin API](http://google.github.io/android-gradle-dsl/javadoc/) What methods/variables are available when extending the plugin

What opened my eyes on how to get this done are the [Android Dev Summit 2015](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc_Tt7q77qwyKRgytF1RzRx8) talks which are **GREAT**. If you didn't watch them already, I strongly suggest you do. The Android Tools Team talked about this function: {% highlight groovy %}variant.registerJavaGeneratingTask(task, outputFolder){% endhighlight %}. Exactly what I needed! From there, parsing the manifest to generating the helper class was just a matter of time & tears.

I hope this plugin will be helpful to more people than myself, and will be glad to get feedback!

