---
layout: post
title:  "How to setup Dagger on Android"
tags: ["kotlin", "dagger", "android", "di", "dependency-injection"]
categories: ["Dependency Injection", "Dagger"]
mermaid: true
comments: true
---

![Dagger](/assets/img/header-dagger-space.png)

Dagger is a dependency injection framework for Java and Android. It's a compile-time framework that generates code for us, making it easier to manage dependencies in our projects.

Let's see how to set it up on Android.

## Dependencies

Add the Dagger dependencies to your `build.gradle` file:

```gradle
dependencies {
    implementation 'com.google.dagger:dagger:2.38.1'
    kapt 'com.google.dagger:dagger-compiler:2.38.1'
} 
```

## Interesting related docs

[Android - Dagger Basics](https://developer.android.com/training/dependency-injection/dagger-basics)

