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

The dependency between **client1** and **sdk** is defined by adding a [Gradle Dependency](https://docs.gradle.org/current/userguide/artifact_dependencies_tutorial.html) to another local project (a.k.a. module) in **client1** to **sdk**:

{% highlight groovy %}
group 'il.co.galex'
version '0.0.1'

...

dependencies {
    testCompile group: 'junit', name: 'junit', version: '4.12'
    compile project(':sdk') // local dependency to the module named "**sdk**"
}
{% endhighlight %}

If we have multiple clients which share our **sdk** we will create them as we created **client1** with the same dependency to **sdk**, side by side:

![monorepo multiple clients]({{ site.url }}/assets/gradle-composite-builds/monorepo-mutiple-clients.png){: .center-image }

The biggest advantage is that you can really quickly make a change on **sdk** and check it immediately in **client1**. You see those two modules side by side under the same project, after all.

This way of organizing our code is good enough when
- You are the sole developer in your company
- All clients need to be maintained forever and you are 100% sure about this
- You don't develop an SDK you need to share with other companies
- You don't plan to share any part of your project via open source

This could be good enough for the moment but to get more elasticity in your work flow the next step is to have multiple projects.

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

The **client1** project needs to get the built **sdk** library from somewhere. This somewhere is called a **Maven repository** and it can be on your local machine or on a distant one like [jCenter](https://bintray.com/bintray/jcenter) or a [Mvn Repository](https://mvnrepository.com/) or even a private [Artifactory](https://www.jfrog.com/artifactory/) behind a VPN, etc.

To learn about how we to deploy a library to a Maven Repository, check this [Gradle Publish Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html).

This structure has two big advantages over one big repository:

- **Proper control versioning**: Having separate projects mean you get a complete overview of the commits per project. It is easier to see what was changed then.

- **Proper library versioning**: When working on **client1** and upgrading the **sdk** to a new version by adding a new feature, we will deploy a new version to our Maven Repository. This means we are **not** obligated to upgrade right now on any other project so we can focus better on our current task.

But this organization has one main problem: the work flow of development. If we leave ourselves with a **client1** and an **sdk** this is what is going to happen every time we will make a change in the **sdk**:

![sdk-work-flow]({{ site.url }}/assets/gradle-composite-builds/sdk-work-flow.png){: .center-image }

Of course, there are two immediate solutions you are probably already thinking about:

- Tests: If the SDK is correctly written, all the features should be well tested and we could work in a more concentrate way by adding multiple features and test them at once then only deploy a new version. True, but good enough.

- A **testapp**: Add an module called **testapp** in our **sdk** project so we can run our library in a client built just for the testing the **sdk**. When the new features are tested, only then deploy a new SDK version for the **client1** project. This generally makes things much easier but we have now an even better solution!

## Why not both?

A new feature was recently added into Gradle and is called Composite Builds. It lets your override an external dependency with a local one, depending on **OUR** terms. Sweet! Let's see how it works.

In a project with a client named **comp-client** in its own project and a library named **comp-sdk** in its own project as well and an external dependency from **client1** to **comp-sdk**, we will declare the following in the file named **settings.gradle** inside **comp-client**:

{% highlight groovy %}
def sdk = '../comp-sdk'
if (file(sdk).exists()) {
    includeBuild(sdk)
}
{% endhighlight %}

We make the assumption the two projects (**comp-client** and **comp-sdk**) sit side by side in our filesystem. We test if **comp-sdk** is actually there besides **comp-client** for real, and if so, include that project as a module. And that's it!
Every change we do in **comp-sdk** will be compiled and used in **comp-client** like if those two are regular submodules!

![composite-builds-in-action]({{ site.url }}/assets/gradle-composite-builds/compsite-builds-in-action.png){: .center-image }


This is **BIG**. It means we get all advantages of both the submodules and the multi-projects worlds.

The only issue here is to not forget to still release a new version of **comp-sdk** to not break your CI Server. A nice [git-hook](https://git-scm.com/book/gr/v2/Customizing-Git-Git-Hooks) could be used to automate the annoying workflow to upgrade+deploy+use a new version but that's worth its own blogpost.

## Conclusion

Both submodules and multi-projects have cons and pros and I think Composite Builds were the missing piece between those two extremes. Composite builds let you have the advantages of multi-projects but without the perpetual context switching between projects and releasing a version for every small change.

Give it a try and/or comment here to tell me what you think about it!
