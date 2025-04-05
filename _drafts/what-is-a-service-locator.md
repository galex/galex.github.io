---
layout: post
title:  "What is a Service Locator"
tags: ["kotlin", "library", "android", "service-location", "DI"]
categories: ["Dependency Injection", "Service Locator"]
mermaid: true
comments: true
---

![A man locates a place on map to show us that place](/assets/img/header-service-locator.png)

What is a Service Locator? Let's dive into it!

## What is a Service

First, let's define what a **Service** is, and no, we're not talking about [Android Services](https://developer.android.com/develop/background-work/services) but more generally about a **Service** in the context of software development.

A **Service** is a piece of code that provides a **specific functionality** to other pieces of code. It can be a class or an interface that has one or more functions with one or more implementations. It can be stateless or stateful (aka keeping variables in memory), if needed. 

We'll take in this post the same examples I usually take, meaning a `Logger` service and a `Network` service.

## What is a Service Locator

A service Locator is actually a [design pattern](https://en.wikipedia.org/wiki/Service_locator_pattern#:~:text=The%20service%20locator%20pattern%20is,to%20perform%20a%20certain%20task.) that provides a centralized registry of services. It is a singleton that knows how to locate services by their name or type.

## Implementation of a Service Locator

Let's implement a simple Service Locator in Kotlin:

```kotlin
interface ServiceLocator {
    fun <T: Any> get(serviceName: String): T
    fun <T: Any> register(serviceName: String, service: T)
}
```
Its implementation looks usually like this:

```kotlin
class ServiceLocatorImpl : ServiceLocator {
    private val services = mutableMapOf<String, Any>()

    override fun <T: Any> getService(serviceName: String): T {
        @Suppress("UNCHECKED_CAST")
        return services[serviceName] as T
    }

    override fun <T : Any> registerService(serviceName: String, service: T) {
        services[serviceName] = service
    }
}
```
That's it! We have a simple Service Locator that can register and retrieve services by their name.

To use a Service Locator, you need to register your services somewhere first, so let's define a `Services` Singleton that will be our entry point to get services:

```kotlin
object Services {
    // We need to create a new instance of ServiceLocatorImpl to register the services
    private val impl = ServiceLocatorImpl().apply {
        // Registering services goes here
    }

    // Helpers to get Services by their name goes here
}
```
## Creating a Logger Service

Let's define a `Logger` interface:

```kotlin
interface Logger {
    fun log(message: String)
}
```
And a `LoggerImpl` implementation:
```kotlin
class LoggerImpl : Logger {
    override fun log(message: String) {
        Log.d("MainLogger", message)
    }
}
```
## Adding the Logger Service to the Service Locator

Now, we'll register those in our `Services` Singleton:

```kotlin
object Services {
    // We need to create a new instance of ServiceLocatorImpl to register the services
    private val impl = ServiceLocatorImpl().apply {
        // Registering a new instance of Logger Service
        register(Logger::class.java.simpleName, LoggerImpl())
    }

    // Getting the Logger service
    fun logger(): Logger = impl.get(Logger::class.java.simpleName)
}
```

This code is kind of ugly, let's improve it thanks to Kotlin Generics and the `reified` keyword:
```kotlin
inline fun <reified T : Any> ServiceLocator.get(): T {
    return get(T::class.java.simpleName)
}

inline fun <reified T : Any> ServiceLocator.register(service: T) {
    register(T::class.java.simpleName, service)
}
```

Using those, our `Services` Singleton looks like this:
```kotlin
object Services {

    private val impl = ServiceLocatorImpl().apply {
        register<Logger>(LoggerImpl())
    }

    fun logger(): Logger = impl.get()
}
```
Cute, isn't it? 😊

Now that we have a `Logger`, in any part of our app, like an Activity for example, we can get the `Logger` service like this:
```kotlin
class MainActivity : ComponentActivity() {

    private val logger: Logger = Services.logger()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        logger.log("onCreate")
    }
}
```
## Creating a Network Service

Now let's continue by adding a `Network` Service that needs a `Logger` service to log messages:

```kotlin
typealias Response = String

interface Network {
    fun fetch(url: String): Response
}
```

And a `NetworkImpl` implementation, that will need to receive the `Logger` Service in its constructor:
```kotlin
class NetworkImpl(
    private val logger: Logger,
) : Network {
    override fun fetch(url: String): Response {
        logger.log("fetch")
        return "fetched data"
    }
}
```

## Adding the Network Service to the Service Locator

Let's add the `Network` service to our `Services` Singleton, like we did for the `Logger` service:

```kotlin
object Services {
    
    private val impl = ServiceLocatorImpl().apply {
        register<Logger>(LoggerImpl())
        register<Network>(NetworkImpl(logger = get()))
    }

    fun logger(): Logger = impl.get()
    fun network(): Network = impl.get()
}
```
Thanks to the reified generic functions we added, we can just call `get()` without any parameters to get the service we need.

## Issues with Service Locator





## Conclusion

