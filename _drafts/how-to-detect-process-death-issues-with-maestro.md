---
layout: post
title:  "How to detect Process Death issues with Maestro"
tags: ["android", "process death", "detection", "reproduction", "find out"]
categories: ["Android"]
mermaid: true
---

## Introduction


## Using Maestro

[Maestro](https://github.com/mobile-dev-inc/maestro) is a surprisingly easy tool to write end-to-end tests and is usually a tool you'd want your Continuous Integration server (GitHub Actions, Bitrise, etc.) to run periodically.

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




