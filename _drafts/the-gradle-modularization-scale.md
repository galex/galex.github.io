---
layout: post
title:  "The Gradle Modularization Scale"
tags: ["kotlin", "library", "android", "gradle", "multiplatform"]
categories: ["Modularization", "Gradle"]
mermaid: true
comments: true
---

![Kardashev Scale](/assets/img/header-kardashev-scale.png)

As a Science Fiction fan, I always loved the idea of the **Kardashev Scale**. 

It's a method of measuring a civilization's level of technological advancement based on the amount of energy they are able to use, divided into three types: The energy from their own planet, their own star, or their own galaxy.

With [Gradle](https://gradle.org/), we can also categorize the modularization level of our project into three types as well! 🤯

## Terminology

First, while talking about Gradle and modularization, there are two main terms that are important to understand:

- **Gradle Project**: A Gradle project is the root project of your build. Its basically just a folder with a `settings.gradle.kts` and a `build.gradle.kts` in it
- **Gradle Subproject**: A Gradle subproject is a project that is part of a larger Gradle project. It is a folder with only a `build.gradle` or `build.gradle.kts` in it

I will mainly use Gradle used with Android as an example, but the concepts are applicable to any Gradle project, pure Java or Kotlin projects or even Kotlin Multiplatform projects.

## Type I

When we create a new project, we get by default a type I project, meaning we have one `settings.gradle.kts` file and one `build.gradle.kts` file in the root of the project, with one `app` folder in it having its own `build.gradle.kts` file, making it a Gradle subproject or more commonly called a module.

```text
  |-app
  |  |-build.gradle.kts // <- The build file for the app module
  |  |-src 
  |     |-main 
  |-build.gradle.kts // <- The build file for the root project
  |-settings.gradle.kts // <- The settings file for the root project
```

For small apps with short development life spans (1 to 3 months), this is completely fine but as the project grows, it becomes harder to maintain and scale a Type I project for many reasons:

- **Build Time**: The more code we have in the app module, the longer the build time will be
- **Testing**: It becomes harder to test code in isolation
- **Code Ownership**: It becomes harder to assign code ownership to teams or individuals
- **Spaghetti Code**: Some devs might take shortcuts and mix code that should be separated

## Type II

Instead of having all the code in one module, we'll divide in into multiple modules, each with its own `build.gradle.kts` file.

```text
  |-app
  |  |-build.gradle.kts // <- The build file for the app module
  |  |-src 
  |     |-main 
  |-architecture
  |  |-build.gradle.kts // <- The build file for the architecture module
  |  |-src 
  |     |-main 
  |-network
  |  |-build.gradle.kts // <- The build file for the network module
  |  |-src 
  |     |-main  
  |-logger
  |  |-build.gradle.kts // <- The build file for the logger module
  |  |-src 
  |     |-main  
  |-dashboard
  |  |-build.gradle.kts // <- The build file for the dashboard module
  |  |-src 
  |     |-main 
  |-settings
  |  |-build.gradle.kts // <- The build file for the dashboard module
  |  |-src 
  |     |-main  
  |-build.gradle.kts // <- The build file for the root project
  |-settings.gradle.kts // <- The settings file for the root project
```

Imagine if we had a project with many modules, the root folder would become very messy and hard to navigate.

To fix that, all modules can be divided into 4 categories:

- **Libraries**: Any toolboxes, utilities, base classes or set of extensions that can be reused across the project
- **Services**: Modules that are responsible for one thing only and can be used by other modules via an interface
- **Features**: Modules containing the UI or the full feature of the app
- **Apps**: Modules that are responsible for the final app

Applied to our Type II example project, we'll have the following hierarchy:

```text
|-apps
|  |-app
|  |  |-build.gradle.kts // <- The build file for the app module
|  |  |-src
|  |     |-main
|-libraries
|  |-architecture
|  |  |-build.gradle.kts // <- The build file for the architecture module
|  |  |-src
|  |     |-main
|  |-network
|  |  |-build.gradle.kts // <- The build file for the network module
|  |  |-src
|  |     |-main
|-services
|  |-logger
|  |  |-build.gradle.kts // <- The build file for the logger module
|  |  |-src
|  |     |-main
|-features
|  |-dashboard
|  |  |-build.gradle.kts // <- The build file for the dashboard module
|  |  |-src
|  |     |-main
|  |-settings
|  |  |-build.gradle.kts // <- The build file for the settings module
|  |  |-src
|  |     |-main
|-build.gradle.kts // <- The build file for the root project
|-settings.gradle.kts // <- The settings file for the root project
```
To go back to our modularization goals, we have now:

- **Build Time**: Thanks to caching, the build time is now faster and we build only what we're changing
- **Testing**: It's easier to test code in isolation as we test only what the module contains
- **Code Ownership**: We can assign code ownership to teams or individuals

## Type II + Build Logic

### Type III

> TL;DR: Many Gradle Projects, Many Gradle Subprojects

Now that's where things get very interesting

If you suffer from a very long build time, you can divide your project into multiple projects, each with its own `settings.gradle.kts` and `build.gradle.kts` files.


## More about Gradle at Scale

- [Gradle's User Guide](https://docs.gradle.org/current/userguide/userguide.html)
- [Gradle for Big Scale Projects]()
- [Google IO 2013 - Android adopts Gradle](https://www.youtube.com/watch?v=LCJAgPkpmR0&ab_channel=GoogleforDevelopers)
- [Google IO 2019 - Build bigger, better: Gradle for large projects](https://www.youtube.com/watch?v=sQC9-Rj2yLI&t=2s&ab_channel=AndroidDevelopers)
- [Slack's Gradle Plugin - Lots to learn from there](https://slackhq.github.io/slack-gradle-plugin/)

## Conclusion

In short: 

- Type I: One Only Gradle Project, One Only Gradle Subproject
- Type II: One Only Gradle Project, Many Gradle Subprojects
- Type II-B: One Main Gradle Project, One build-logic Gradle Project, Many Gradle Subprojects
- Type III: Many Gradle Projects, Many Gradle Subprojects

By the way, I'm also on [Twitter](https://twitter.com/galex) and [LinkedIn](https://www.linkedin.com/in/agherschon/), so feel free to connect there too!

Happy modularization! 🚀
