---
layout: post
title: "[WIL] Watching Droidcon NYC 2016"
date: "2016-11-26 15:56:57 +0200"
---

Hello,

In this post I will summarize what I've learned from the talks of **[Droidcon NYC 2016](https://www.youtube.com/playlist?list=PLnVy79PaFHMXJha06t6pWfkYcATV4oPvC)**.

### Opening keynote: android is for everyone ###
by @kellyShuster



### Deep Dive Into the Gradle-Based Android Build ###
by [Hans Dockter](https://www.linkedin.com/in/hansdockter), founder of Gradle

* [Multi Repo support with Composite Builds](https://docs.gradle.org/current/userguide/composite_builds.html)

  When we are developing a library and we don't want to deploy to our local maven repo on every change, we can now use a composite build pointing directly to our other folder for a dependency. Sweet!

- Cached Builds

  A task is said up-to-date if nothing changed from **last time** when we ran **this build** on **this machine**. With a **new local cache**, it becomes now up-to-date if nothing changed from **anytime before** when we ran **any build** on this machine. An with a **distributed cache**, a task is up-to-date is nothing changed from **anytime before** when we ran **any build anywhere**. Anything to win build time is for me a good step in the right direction.  

- Build Scans

  Centralized way to share a build report, and in the future we will have comparisons (enterprise feature).

  ![alt text](http://i.imgur.com/0UJH96om.png "Build scan")




## Conclusion ##
Thank you touchlab to record and share all those amazing talks!
