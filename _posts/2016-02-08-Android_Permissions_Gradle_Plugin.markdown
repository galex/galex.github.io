---
layout: post
title: Android Permissions Gradle Plugin
category: android dev gradle plugin permissions marshmallow
---

My first Gradle Plugin, [Android Permissions Gradle Plugin](https://github.com/galex/android-permissions-gradle-plugin) was just deployed into jCenter. 

Developping this was a great experience. I use gradle heavily at work, of course, as the company has a project with lots of flavors. But the need never arose for a Gradle Plugin until recently, where I wanted to compile the SDK with the latest Android version, 23.

What is the issue with the new Android Permissions System in Marshmallow? Nothing, it's a great feature. It was long time due in my opinion. You have to request on Runtime the dangerous permissions you have declared in your Android Manifest. But the issue appears if you want to request, let's say, all the dangerous permissions you require at the start of the application, you'll have to maintain a second list of dangerous permissions yourself somewhere, because there is no method to request *all the dangerous permissions* from the Android Manifest (that I'm aware of).

But what happens when you start to play with different set of dangerous permissions between flavors? Let's say Flavor A requires two dangerous permissions, and Flavor B requires only one. You can see here the problem. This plugin does just that: it parses the Manifest for each **Variant** and generates its **PermissionsHelper**, which has the list of dangerous permissions as a static attribute plus a few convenient methods to help with requesting and checking if a permission was granted, or not.

So what this plugin needs to do? Parse the Manifest per variant and generate some Java Code in a folder Gradle will use as a source. I had no idea where to start, and the documentation is a little ruff and difficult to find, not that it's not there, but it's not clear really clear what you need: Gradle doc or API? Android Gradle plugin docs? DSL? or maybe API? I just want to say something about this: the Android Tools team does an amazing job at improving our tools, so I can understand the doc is a little hard to access for gradle beginners. I am sure they will improve it overtime, but for now here is a compilation of links about it:

- [Gradle User Guide](https://docs.gradle.org/current/userguide/userguide.html): How to use gradle components in general
- [Gradle DSL API](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html): What variables are available in Gradle for each component?

So fast forward to last wweekend when I decided finally to catch up on the [Android Dev Summit 2015](https://www.youtube.com/playlist?list=PLWz5rJ2EKKc_Tt7q77qwyKRgytF1RzRx8), where I watched 18 talks on one day. You know, it's like a TV show, you start one, and the next one starts to play automatically, and you're like "Eh why not another one?". Those talks are **GREAT**. If you didn't watch them already, I strongly suggest you do now.




