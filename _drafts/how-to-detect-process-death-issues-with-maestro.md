---
layout: post
title:  "Detecting Process Death issues with Maestro"
tags: ["android", "process death", "detection", "reproduction", "find out", "appium", "automation", "qa", 'maestro']
categories: ["Android", "Process Death"]
mermaid: true
comments: true
---

![Maestro playing the flute to a serpent](/assets/img/header-maestro-serpent.png)

After [explaining](/posts/process-death-is-the-rule-not-the-exception/) what **System-initiated Process Death** is, showing how to [detect it manually](/posts/how-to-detect-process-death-issues/) and [automate detection with Appium](/posts/detecting-process-death-issues-with-appium/), I'd like to demonstrate how to automate detection this time with [Maestro](https://maestro.mobile.dev/), a relatively new Automation tool! 🎉

## What is Maestro?

[Maestro](https://maestro.mobile.dev/) is a command line tool that allows us to run end-to-end UI tests, written in a simple YAML format.

Run the following command to install Maestro on Mac OS, Linux or Windows (WSL):

```shell
curl -Ls "https://get.maestro.mobile.dev" | bash
```

## Using Maestro

Let's go back to the [demo project](https://github.com/galex/process-death-demo-project) containing a fragment-based app built with two screens:
- A first screen to enter a name called **EnterNameFragment**
- Second one to show that information called **ShowInfoFragment** 
- 
To pass a name value between those screens, a **Singleton** called **DataHolder** is used, which will lose its value after **System-initiated Process Death**.

A file called `happy-flow.yaml` in a `.maestro` folder will contain the following maestro commands:

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

We'll run this test with the following command, from the demo project root folder:

```shell
maestro test .maestro/happy-flow.yaml
```

The result of running this test will show a successful run for our test:

```shell
Running on 41161FDJG002EG                                                                            
                                                                                                     
 ║                                                                                                   
 ║  > Flow                                                                                           
 ║                                                                                                   
 ║    ✅  Launch app "dev.galex.process.death.demo"                                                  
 ║    ✅  Tap on id: dev.galex.process.death.demo:id/enter_name                                      
 ║    ✅  Input text John Doe                                                                        
 ║    ✅  Tap on id: dev.galex.process.death.demo:id/next                                            
 ║    ✅  Assert that "Name = John Doe", id: dev.galex.process.death.demo:id/show_name is visible    
 ║    ✅  Stop dev.galex.process.death.demo                                                          
 ║   
```

## Triggering The Issue with Maestro

Since version XXX, Maestro offers the ability to trigger **System-initiated Process Death** using a command called `killApp`, which didn't exist before, and gave me the opportunity [to contribute to the Maestro project](https://github.com/mobile-dev-inc/maestro/pull/1727). 🎉

We'll call this file appropriately `detect-process-death-issue.yaml`:

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
- pressKey: Home
- killApp
- launchApp:
    stopApp: false
- assertVisible:
    id: "dev.galex.process.death.demo:id/show_name"
    text: "Name = John Doe"
    label: "Assert that \"Name = John Doe\" meaning the screen kept its data after System-initiated Process Death"
    enabled: true
- stopApp
```
Let's pay attention to the part that helps us reproduce the issue, with explanations in comments:
```yaml
- pressKey: Home # Puts the app into the background
- killApp # Kills the app (adb shell am kill)
- launchApp: # Relaunches the app
    stopApp: false # Without adb shell am stop
```
We'll run this test with the following command:
```shell
maestro test .maestro/detect-process-death-issue.yaml
```
The result of running this test will fail:
```shell
Running on 41161FDJG002EG                                                                                        
                                                                                                                 
 ║                                                                                                               
 ║  > Flow                                                                                                       
 ║                                                                                                               
 ║    ✅  Launch app "dev.galex.process.death.demo"                                                              
 ║    ✅  Tap on id: dev.galex.process.death.demo:id/enter_name                                                  
 ║    ✅  Input text John Doe                                                                                    
 ║    ✅  Tap on id: dev.galex.process.death.demo:id/next                                                        
 ║    ✅  Assert that "Name = John Doe", id: dev.galex.process.death.demo:id/show_name is visible                
 ║    ✅  Press Home key                                                                                         
 ║    ✅  Kill dev.galex.process.death.demo                                                                      
 ║    ✅  Launch app "dev.galex.process.death.demo" without stopping app                                         
 ║    ❌  Assert that "Name = John Doe" meaning the screen kept its data after System-initiated Process Death    
 ║    🔲 Stop dev.galex.process.death.demo                                                                       
 ║                                                                                                               
                                                                                                                 
Assertion is false: "Name = John Doe", id: dev.galex.process.death.demo:id/show_name is visible

==== Debug output (logs & screenshots) ====

/Users/galex/.maestro/tests/2024-06-07_092526
```
Perfect! We've successfully detected the **System-initiated Process Death** issue with Maestro! 🎉

In the generated folder `/Users/galex/.maestro/tests/2024-06-07_092526` we'll find the screenshots taken by Maestro, showing the **ShowInfoFragment** screen with the text `Name = null` instead of `Name = John Doe`:

![Faulty screen](/assets/img/screenshot-❌-1717741567602-(detect-process-death-issue.yaml).png)

## Improving our Maestro Test

To be able to re-use the commands that allow us to trigger **System-initiated Process Death**, we can put them in another `.yaml` file named `trigger-process-death.yaml`:
```yaml
appId: dev.galex.process.death.demo
---
- pressKey: Home # Puts the app into the background
- killApp # Kills the app (adb shell am kill)
- launchApp: # Relaunches the app
    stopApp: false # Without adb shell am stop
```
And use the `runFlow` command in our original `detect-process-death-issue.yaml` test to include it:
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
- runFlow: trigger-process-death.yaml
- assertVisible:
    id: "dev.galex.process.death.demo:id/show_name"
    text: "Name = John Doe"
    label: "Assert that \"Name = John Doe\" meaning the screen kept its data after System-initiated Process Death"
    enabled: true
- stopApp
```
The output of running `detect-process-death-issue.yaml` will look almost the same as before:
```shell
Running on 41161FDJG002EG                                                                                        
                                                                                                                 
 ║                                                                                                               
 ║  > Flow                                                                                                       
 ║                                                                                                               
 ║    ✅  Launch app "dev.galex.process.death.demo"                                                              
 ║    ✅  Tap on id: dev.galex.process.death.demo:id/enter_name                                                  
 ║    ✅  Input text John Doe                                                                                    
 ║    ✅  Tap on id: dev.galex.process.death.demo:id/next                                                        
 ║    ✅  Assert that "Name = John Doe", id: dev.galex.process.death.demo:id/show_name is visible                
 ║    ✅  Run trigger-process-death.yaml                                                                         
 ║    ❌  Assert that "Name = John Doe" meaning the screen kept its data after System-initiated Process Death    
 ║    🔲 Stop dev.galex.process.death.demo                                                                       
 ║                                                                                                               
                                                                                                                 
Assertion is false: "Name = John Doe", id: dev.galex.process.death.demo:id/show_name is visible                  
                                                                                                                 
==== Debug output (logs & screenshots) ====                                                                      

/Users/galex/.maestro/tests/2024-06-07_105906
```
But we've improved the reusability of our detecting Process Death Flow!

## Conclusion

We've successfully detected the **System-initiated Process Death** issue with [Maestro](https://github.com/mobile-dev-inc/maestro)! 💪 It makes the whole ordeal so easy I seriously think [Maestro](https://github.com/mobile-dev-inc/maestro) should be a part of every feature we build to ensure its quality and robustness! 🚀

Let me know what you think or if you have any questions in the comments! 📝

Happy testing! 🧪



