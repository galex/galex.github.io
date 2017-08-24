---
layout: post
title: Introduction to Gradle
permalink: Introduction-to-Gradle
---

Gradle is a build system: its goal is to build your project. To install Gradle:
{% highlight bash %}
brew update && brew install gradle
{% endhighlight %}

Gradle revolves around main two concepts:

## Gradle Tasks

A task is something Gradle needs to run in order to build your project. Let’s create a file named **“hello.gradle”** and write the following code in it:
{% highlight groovy %}
task(hello) {
        doLast {
                println "Hello World!"
        }
}
{% endhighlight %}

To run the task open your favorite console app and run the following:

{% highlight bash %}
gradle -b hello.gradle hello
{% endhighlight %}

This will output to:

{% highlight bash %}
> Task :hello
Hello World!
{% endhighlight %}

Congratulations! You just wrote your first gradle task in a gradle script, in [Groovy](http://groovy-lang.org/)!

Gradle comes with [**many tasks**](https://docs.gradle.org/current/userguide/more_about_tasks.html) by default, and to extend Gradle capabilities tasks are packaged into **Gradle Plugins** to offer more specific tasks to your type of project.

## Gradle Plugins

Plugins add tasks to your project to build whatever you want to build.

A Java developer which would want to build his Java project will use a Gradle Plugin called the [Java Gradle Plugin](https://docs.gradle.org/current/userguide/java_plugin.html).

Gradle is not limited to the Java world! An iOS developer could also use Gradle to build his iOS app by using a specific plugin created to build [iOS projects](https://github.com/openbakery/gradle-xcodePlugin). A .Net developer could build his own .Net project by using a specific [.Net Gradle Plugin](https://github.com/Ullink/gradle-msbuild-plugin). You get the idea!

On Android, Google develops the [Android Gradle Plugin](http://tools.android.com/build/gradleplugin). This allows new and existing Android Projects to build applications (.apk) and libraries (.aar).

Let’s see a concrete example. Download [IntelliJ](https://www.jetbrains.com/idea/) and create a basic Gradle based Java project:

![Create a basic gradle Java project]({{ site.url }}/assets/introduction-to-gradle/create-java-gradle-project.png){: .center-image }

In this newly created project, you can see two important files:

![project structure]({{ site.url }}/assets/introduction-to-gradle/project-structure.png){: .center-image }

The **settings.gradle** file is the main entry point for gradle in a folder. In this file, the main project is configured or if you have a multi-modules project, the list of modules will be set. In every project/module, a **build.gradle** file will indicate to Gradle how you intend to build your project/module.

In this case, as you created a Java project, your build.gradle will look like this:

![build-gradle-file]({{ site.url }}/assets/introduction-to-gradle/build-gradle.png){: .center-image }

On line number **4** you can see that the **Java Gradle plugin** is applied. This means a list of Java related tasks were added through this plugin and now we can build a Java project with it!

The most important task you need to learn about is a task named **“tasks”**. This is the first task I will run when I enter an existing project to see what are the tasks that are defined in this project. Run “gradle tasks” inside your newly created project:

{% highlight bash %}
cd my-java-project
gradle tasks
# or use the Gradle wrapper instead
./gradlew tasks
{% endhighlight %}

The **Gradle Wrapper** is an executable script that embeds Gradle inside your project. That way every project on your computer can have its own Gradle version and you don't have to install every Gradle version directly on your computer. Not that you don't have to use a Gradle wrapper but it is considered a good practice to do so.

And the output will look like this:

![available-tasks]({{ site.url }}/assets/introduction-to-gradle/java-gradle-tasks.png){: .center-image }

You can see now that we have a task named **jar** which will compile the Java code and package it inside a Jar file that you will find in the folder **builds/libs** inside your project.

## Conclusion

Gradle is awesome! Everything you do which revolves around your project can be automated into your project Gradle build. For example, at my current job, our team uses daily a Gradle Plugin to download and adapt string translations from a internal company tool the translations team uses.

To continue learning about Gradle you should totally have a look at the [official guides](https://gradle.org/guides/)!
