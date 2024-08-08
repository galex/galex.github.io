---
layout: post
title:  "The missing glue for API/DI: dagger-devek"
tags: ["kotlin", "library", "android", "gradle", "multiplatform"]
categories: ["Modularization", "Gradle"]
mermaid: true
comments: true
---

![Devek means glue](/assets/img/headers-strands-of-glue.png)

The main problem with [API/DI](posts/advanced-modularization-api-impl-vs-api-di/) is the fact that it created boilerplate as the app needs to provide instances between modules.

dagger-devek is a dagger-spi plugin to generate the boilerplate code for us, using naming conventions. 

## What is API/DI



## What is a Dagger SPI Plugin?

## What does dagger-devek do?

## How to use dagger-devek

https://github.com/google/dagger/issues/3464



## Conclusion

In short: 

- Type I: One Only Gradle Project, One Only Gradle Subproject
- Type II: One Only Gradle Project, Many Gradle Subprojects
- Type II-B: One Main Gradle Project, One build-logic Gradle Project, Many Gradle Subprojects
- Type III: Many Gradle Projects, Many Gradle Subprojects

By the way, I'm also on [Twitter](https://twitter.com/galex) and [LinkedIn](https://www.linkedin.com/in/agherschon/), so feel free to connect there too!

Happy modularization! 🚀
