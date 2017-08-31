---
layout: post
title: The Best of Two Worlds with Gradle Composite Builds
category:
---
For a basic understanding of what Gradle is I recommend you read this [Introduction to Gradle](http://galex.co.il/2017/07/24/Introduction-to-Gradle.html) first.

At some point in your Android developer career you will come across a situation where you would like to share code between two or more apps. It could be some utility methods or a new innovative way to do something that is difficult to code right now, and you might even want to share it with the community via open source.

When we have some common code to share between apps the way to do it is to build a **Java library** (**.jar** file) if it has nothing related to Android or as an **Android library** if it does (.aar file). That way our code is [written once](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and can be reused easily. To share our code build in as a library we have two different ways.

## One big project (monorepo)

Usually we will start to separate that library code by putting it in its own [module](https://www.jetbrains.com/help/idea/about-modules.html). Let's suppose we have a project called **client1** and a library called **sdk**

![monorepo with one sdk and client1]({{ site.url }}/assets/gradle-composite-builds/monorepo.png){: .center-image }

This project structure in IntelliJ will look like this:

![monorepo project with one sdk and client1]({{ site.url }}/assets/gradle-composite-builds/monorepo-project.png){: .center-image }

The dependency between **client1** and **sdk** is defined by adding a Gradle Dependency in **client1** to **sdk**:

![monorepo project dependency]({{ site.url }}/assets/gradle-composite-builds/client1-dep-sdk.png){: .center-image }

## Multiple projects



## Why not both?
