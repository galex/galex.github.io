---
layout: post
title:  "Dependency Injection Series"
tags: ["kotlin", "library", "android", "DI", "dependency-injection" , "inversion-of-control", "ioc"]
categories: ["Dependency Injection", "Dagger"]
mermaid: true
comments: true
---

## Introduction

What is Dependency Injection, what concepts are behind it, and what tools are available and what do they offer to us, Android developers? 

> ℹ️ TLDR: [Series of posts about Dependency Injection](#posts)

# Inversion of Control

The [official definition](https://en.wikipedia.org/wiki/Inversion_of_control#Alternative_meaning) can be found here, but in short means that we, developers, understood that creating objects inside our classes is a **very bad idea**. It makes them hard to **test**, hard to **maintain**, and hard especially to **understand**. 


This also connects to the **Single Responsibility Principle**, which states that a class should have only responsibility, to do what is was created to do, and nothing else. 


So, let's first look at a few Dependency Injection frameworks that are available to us, and also even build some our own!

# Feature Requirements

We basically need to be able to [create objects](#creation-of-objects), and to be able to [manage their lifecycle](#lifecycle-of-objects).

## Creation of objects

We would like to be able to get objects that can be

- **New Instances**, aka normal new objects to be used when we need them and disposed when not needed anymore
- **Singletons** (like a Logger Service)
- **Factories** (like a ViewModel Factory, to tell Android how to create a ViewModel with some parameters or dependencies)

## Lifecycle of objects

A Singleton should probably be created only once, and then kept in memory until the end of the application.

# Two types of Dependency Injection frameworks

There are two types of Dependency Injection frameworks:

- Runtime frameworks, like **Koin**
- Compile-time frameworks, like **Dagger2**

# Posts

To answer all those needs and questions into this board topic, I've cut this series into multiple posts:

- [What is a Service Locator](/what-is-service-locator)

- What is a Service Locator and why is it meh?
- What is a Dependency Injection Container and why is it better?

To answer those questions,

