---
layout: post
title:  "How to detect Process Death issues with Appium"
tags: ["android", "process death", "detection", "reproduction", "find out", "appium", "automation", "QA"]
categories: ["Android", "Process Death"]
mermaid: true
---

![Giant robot looking for bugs in a field of flowers](/assets/img/robot-looking-for-bugs.png)

Detecting manually Process Death issues is not easy and definitely time-consuming. 

Plus, a change of navigation or architecture could potentially break our app and even if we took care of those issues before, they could re-surface. Wouldn't it be ideal if we could take care of this once and for all? 

[Appium](http://appium.io/docs/en/latest/) is a end-to-end (e2e) automation tool that is widely used at companies investing in their mobile app properly, so let's check how it can help us detect the [Process Death issue](https://galex.dev/posts/how-to-detect-process-death-issues/) present in our [demo app](https://github.com/galex/process-death-demo-project). 

> ℹ️ This is the **4th** post on Process Death, and I invite you to read the previous ones before:
> 1. [Process Death is the rule, not the exception!](https://galex.dev/posts/process-death-is-the-rule-not-the-exception/)
> 2. [Every Screen is an Entry Point](https://galex.dev/posts/every-screen-is-an-entry-point/)
> 3. [How to detect Process Death issues](https://galex.dev/posts/how-to-detect-process-death-issues/)

## Appium Setup

On Ubuntu, here are the steps to run to set up Appium.

First, install Appium:

```shell
npm install -g appium
```
Then install the driver needed to test Android apps:
```shell
appium driver install uiautomator2
```
Then run the Appium server like this:
```shell
appium --relaxed-security
```
> ⚠️ `--relaxed-security` option is needed to be able to access adb commands directly.
>
> Depending on where your Automation CI runs, this could be problematic 🫤

Be sure to have the apk of the app we're testing on your connected device.

## Appium Demo Project

Writing e2e tests with Appium means writing Kotlin JUnit tests in a JVM Project. 

The project we'll work in to write our Appium automation tests can be [found right there](https://github.com/galex/process-death-demo-appium).

We'll write two tests, one for a basic happy flow that represents what we expect the flow to be used for, and one where we'll reproduce our Process Death issue.

Both tests inherit from a base `open class` named `BaseAppiumTest`:

```kotlin
open class BaseAppiumTest {

  companion object {

    lateinit var driver: AndroidDriver

    @JvmStatic
    @BeforeAll
    fun setUp() {
        
      val options = UiAutomator2Options()
        .setPlatformName(PLATFORM_NAME)
        .setAppPackage(PACKAGE_NAME)
        .setAutomationName(AUTOMATION_NAME)
        .setAppActivity(ACTIVITY_NAME)
        .setAutoGrantPermissions(true)

      driver = AndroidDriver(URL(SERVER_URL), options)
      driver.activateApp(PACKAGE_NAME)
    }

    @JvmStatic
    @AfterAll
    fun tearDown() {
      driver.quit()
    }
  }
}
```
Where the constants are:
```kotlin
const val ACTIVITY_NAME = "dev.galex.process.death.demo.MainActivity"
const val PACKAGE_NAME = "dev.galex.process.death.demo"
const val AUTOMATION_NAME = "UiAutomator2"
private const val PLATFORM_NAME = "Android"
const val SERVER_URL = "http://127.0.0.1:4723"
```

## A basic happy flow
To test the basic flow in this demo app, we will do the following steps:
- Open the app
- Fill in the name **"John Doe"**
- Click on the Next button 
- Expect to see **"Name = John Doe"** in the second screen

Here's how it looks written down in code:
```kotlin
class EnterNameFlow: BaseAppiumTest() {

    @Test
    fun `Basic Happy Flow`() {

        // Finds the TextView in the first screen
        val textView: WebElement = driver.findElement(AppiumBy.id("dev.galex.process.death.demo:id/enter_name"))
        assertNotNull(textView)
        // Fills it up with "John Doe"
        textView.sendKeys("John Doe")

        // Finds the Button "Next" to go to the second screen
        val button = driver.findElement(AppiumBy.id("dev.galex.process.death.demo:id/next"))
        // Clicks on the button
        button.click()

        // Gets the string presented in the ShowName TextView
        val showNameText = driver.findElement(AppiumBy.id("dev.galex.process.death.demo:id/show_name")).text

        // Checks that we really see what we expect in this screen
        assertEquals(showNameText, "Name = John Doe")
    }
}
```
Let's see what happens when we run this test:

![running-basic-happy-flow-appium-test-passes-in-IDE](/assets/img/run-basic-happy-flow-appium-test.png)

It **succeeds**, as **expected**!

The QA Team is happy, the Android devs are happy, we're all happy!

## Adding Process Death Detection

Throwing **Process Death** in the mix means the flow looks like this:
- Open the app
- Fill in the name **"John Doe"**
- Click on the Next button
- **Put the app into the background by pushing the Home Button**
- **Kill the app by `adb shell am kill <package name>`**
- **Launch the app as it was (like coming from Recent Apps)**
- Expect to see **"Name = John Doe"** in the second screen

Here's how it looks like:
```kotlin
class EnterNameProcessDeathFlow : BaseAppiumTest() {

    @Test
    fun `Basic Happy Flow + Process Death`() {

        // Finds the TextView
        val textView: WebElement = driver.findElement(AppiumBy.id("dev.galex.process.death.demo:id/enter_name"))
        assertNotNull(textView)
        // Fills it up with "John Doe"
        textView.sendKeys("John Doe")

        // Finds the Button "next" and clicks on it
        val button = driver.findElement(AppiumBy.id("dev.galex.process.death.demo:id/next"))
        // Clicks on the button
        button.click()

        // Puts app in background
        driver.pressKey(KeyEvent(AndroidKey.HOME))
        // Wait for a bit
        Thread.sleep(1_000)
        // Kills the app via adb shell am kill
        driver.killApp()
        // Wait a bit
        Thread.sleep(1_000)
        // Starts the app via adb shell am start
        driver.restart()
        // Wait a bit
        Thread.sleep(1_000)

        // Gets the string presented in the ShowName TextView
        val showNameText = driver.findElement(AppiumBy.id("dev.galex.process.death.demo:id/show_name")).text

        // Checks that we really see what we expect in this screen
        assertEquals("Name = John Doe", showNameText)
    }
```
The `killApp()` extension function is using `driver.executeScript` to be able to call `adb shell am kill`:
```kotlin
fun AndroidDriver.killApp() {
  
    val args = listOf("kill", PACKAGE_NAME)
    val command = mapOf(
        "command" to "am",
        "args" to args
    )

    executeScript("mobile: shell", command)
}
```
To relaunch the app, we can't use `terminateApp()` of the `AndroidDriver` interface because it uses a `force-stop` internally, so an alternative is to call `adb shell am start` directly:
```kotlin
fun AndroidDriver.restart() {

    val args = listOf("start", "$PACKAGE_NAME/$ACTIVITY_NAME")
    val command = mapOf(
        "command" to "am",
        "args" to args
    )

    executeScript("mobile: shell", command)
}
```
Running this test gives us the following result:

![running basic happy flow with process death appium test fails in IDE](/assets/img/run-basic-happy-flow-process-death-appium-test.png)

It **fails**, as **expected**!

The QA Team is happy, the Android devs are happy it got caught, and we can fix it, we're all happy!

## Conclusion

We've **automated** detecting **Process Death issues** using Appium!

I'm still not done on the subject, stay tuned!
