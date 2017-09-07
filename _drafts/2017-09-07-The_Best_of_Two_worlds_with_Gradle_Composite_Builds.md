---
layout: post
title: The Best of Two Worlds with Gradle Composite Builds
category:
---
For a basic understanding of what Gradle is I recommend you read this [Introduction to Gradle](http://galex.co.il/2017/07/24/Introduction-to-Gradle.html) first.

At some point in your developer career you will come across a situation where you would like to share code between two or more apps. It could be some utility methods or a super cool annotation-based database library and you might even want to share it with the community by open sourcing this piece of well crafted code.

When we have some common code to share between apps the way to do it is to build a **Java library** (**.jar** file) if it has nothing to do with Android or as an **Android library** if it does (**.aar** file). That way our code is [written once](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) and can be reused easily. To share our code as a library we have two different ways.

## One big project with submodules

We will start to separate the code meant to be a library by putting it in its own [module](https://www.jetbrains.com/help/idea/about-modules.html). Let's suppose we have a project called **client1** and a library called **sdk**

![monorepo with one sdk and client1]({{ site.url }}/assets/gradle-composite-builds/monorepo.png){: .center-image }

This project structure in IntelliJ will look like this:

![monorepo project with one sdk and client1]({{ site.url }}/assets/gradle-composite-builds/monorepo-project.png){: .center-image }

The dependency between **client1** and **sdk** is defined by a [Gradle Dependency](https://docs.gradle.org/current/userguide/artifact_dependencies_tutorial.html) to another local project (a.k.a. module) in **client1** to **sdk**:

{% highlight groovy %}
group 'il.co.galex'
version '0.0.1'

...

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile project(':sdk') // local dependency to the module named "sdk"
}
{% endhighlight %}

If we have multiple clients which share our **sdk** we will create them as we created **client1** with the same dependency to **sdk**, side by side:

![monorepo multiple clients]({{ site.url }}/assets/gradle-composite-builds/monorepo-mutiple-clients.png){: .center-image }

The biggest advantage here is that you can really quickly make a change on **sdk** and run immediately **client1**. You see those two modules side by side under the same project, all its code is right in front of you.

This way of organizing our code is good enough when
- You are the sole developer in your company
- All clients need to be maintained forever and you are 100% sure about this
- You don't develop an SDK you need to share with other companies
- You don't plan to share any part of your project via open source

This could be good enough for a while but to get more flexibility in your day to day work flow the next step is to have multiple projects.

You should know there's an option to use [git submodules](https://git-scm.com/docs/git-submodule) (or [svn externals](http://svnbook.red-bean.com/en/1.0/ch07s03.html)) to share modules between projects. This is a valid solution but it is not as comfortable as having multiple projects, and I find this solution quite annoying especially when you use branching quite a lot.

## Multiple projects

Every module we had before (**client1** and **sdk**) will move in its own project:

![multi-projects]({{ site.url }}/assets/gradle-composite-builds/multi-projects-simple.png){: .center-image }

We will declare a [Gradle Dependency](https://docs.gradle.org/current/userguide/artifact_dependencies_tutorial.html) to **sdk** like we add any 3rd-party library:

{% highlight groovy %}
dependencies {
    compile "il.co.galex:sdk:0.0.1" // external library
}
{% endhighlight %}

But what is really happening when you synchronize Gradle after you added this dependency:

![multi-projects-with-maven-repo]({{ site.url }}/assets/gradle-composite-builds/multi-projects-maven-repos.png){: .center-image }

The **client1** project needs to get the built **sdk** library from somewhere. This somewhere is called a **Maven repository** and it can be on your local machine, on a distant one like [jCenter](https://bintray.com/bintray/jcenter) or a [Mvn Repository](https://mvnrepository.com/) or a private [Artifactory](https://www.jfrog.com/artifactory/) behind a VPN.

To learn about how we to deploy a library to a Maven Repository, check this [Gradle Publish Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html).

This structure has two big advantages over one big repository:

- **Proper control versioning**: Having separate projects mean you get a complete overview of the commits per project. It is easier to see what was changed and it is so much easier to work with branching when needed.

- **Proper library versioning**: When working on **client1** and upgrading the **sdk** to a new version by adding a new feature, we will have to deploy a new version to our Maven Repository. This means we are **not** obligated to upgrade right now any other project so we can better focus on our current task.

This organization has one main problem: the development work flow. If we leave ourselves with a **client1** and an **sdk** this is what is going to happen every time we will make a change in the **sdk**:

![sdk-work-flow]({{ site.url }}/assets/gradle-composite-builds/sdk-work-flow.png){: .center-image }

Of course, there are two immediate solutions you are probably already thinking about:

- **Tests**: If the SDK is correctly written, all the features should be well tested and we could work in a more concentrate way by adding multiple features and test them at once then only deploy a new version. True, but tests to not replace running the software itself.

- A **testapp**: We'll add a module called **testapp** in our **sdk** project so we can run our library in a client built just for the sake of running the **sdk**. It's a nice solution, as if we had to develop an new UI component on Android, we could add it in a screen multiple times in different sizes to check that it behaves correctly. This way of checking a library was the best we had for a long time, but now we have a better solution!

## Why not both?

[Composite Builds](https://docs.gradle.org/current/userguide/composite_builds.html) is a new feature (added in Gradle since version 3.5) which lets your override an external dependency with a local one, depending on **OUR** terms. Sweet! Let's see how it works.

In a project with a client named **comp-client** in its own project and a library named **comp-sdk** in its own project as well and an external dependency from **comp-client** to **comp-sdk**, we will declare the following in the file named **settings.gradle** inside **comp-client**:

{% highlight groovy %}
def sdk = '../comp-sdk'
if (file(sdk).exists()) {
    includeBuild(sdk)
}
{% endhighlight %}

We make the assumption the two projects (**comp-client** and **comp-sdk**) sit side by side in our filesystem. We test if **comp-sdk** is actually there besides **comp-client** for real, and if so, include that project as a module.

And that's it! Every change we make in **comp-sdk** will be compiled and used in **comp-client** like if those two are regular submodules!

![composite-builds-in-action]({{ site.url }}/assets/gradle-composite-builds/compsite-builds-in-action.png){: .center-image }

This is **BIG**. It means we get all advantages of both submodules and multi-projects worlds, together!

This feature looks for a matching name between a module and the artifact id of the external dependency. If it happens that those do not match, you can explicitely define which dependency it replaces:

{% highlight groovy %}
def sdk = '../comp-sdk'
if (file(sdk).exists()) {
    includeBuild(sdk) {
        dependencySubstitution {
            substitute module('il.co.galex:other-art-name') with project(':')
        }
    }
}
{% endhighlight %}

Thanks to the if-condition, if one of our colleagues still prefers the multi-projects setting she/he is free to continue to do so by simply not having **comp-sdk** be side by side in her/his filesystem.

The only remaining issue is to not forget to still release a new version of **comp-sdk** at the end of the day for the people or CI servers who will not use this feature. A nice [git-hook](https://git-scm.com/book/gr/v2/Customizing-Git-Git-Hooks) could be used to automate the annoying workflow of upgrading/deploying/consuming a new version but that's worth its own blogpost.

## Conclusion

Both submodules and multi-projects have pros and cons. [Composite Builds](https://docs.gradle.org/current/userguide/composite_builds.html) are the perfect middle ground. [Composite Builds](https://docs.gradle.org/current/userguide/composite_builds.html) give us the advantages of submodules by having all the code under the same project easily modifiable and quickly runnable. We also get the advantages of multi-projects of versioning and isolation without the perpetual context switching between projects and releasing a version for every small change.

Give it a try and comment here to tell me what you think about it!
