---
layout: post
title:  "Detecting Process Death issues with Maestro"
tags: ["android", "process death", "detection", "reproduction", "find out", "appium", "automation", "QA"]
categories: ["Android", "Process Death"]
mermaid: true
---

![Giant robot looking for bugs in a field of flowers](/assets/img/header-green-robot.png)

Detecting System-initiated Process Death issues is difficult and time-consuming!

It would be **so great** if we had a way to **automate** checking for those issues so that we'll free ourselves all this time.

> ℹ️ This is the **4th** post on Process Death, and I invite you to read the previous ones before:
> 1. [Process Death is the rule, not the exception!](https://galex.dev/posts/process-death-is-the-rule-not-the-exception/)
> 2. [Every Screen is an Entry Point](https://galex.dev/posts/every-screen-is-an-entry-point/)
> 3. [How to detect Process Death issues](https://galex.dev/posts/how-to-detect-process-death-issues/)
> 4. [How to detect Process Death Issues with Appium]()


## Using Maestro

[Maestro](https://github.com/mobile-dev-inc/maestro) is quite new compared to [Appium](http://appium.io/docs/en/latest/) and is surprisingly easy tool to write end-to-end tests and is usually a tool you'd want your Continuous Integration server (GitHub Actions, Bitrise, etc.) to run periodically.

Our current flow can be tested like this:

```yaml
appId: dev.galex.process.death.demo
---
- launchApp
- tapOn:
    id: "dev.galex.process.death.demo:id/enter_name"
- inputText: "John Doe"
- tapOn:
    id: "dev.galex.process.death.demo:id/next"
- assertVisible:
    id: "dev.galex.process.death.demo:id/show_name"
    text: "Name = John Doe"
    enabled: true
- stopApp
```

## Conclusion


Android Developers do need to allocate time on their tasks to test and trigger System-initiated Process Death to be certain their screens actually work properly.




