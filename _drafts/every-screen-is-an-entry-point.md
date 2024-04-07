---
layout: post
title:  "Every Screen is an Entry Point"
tags: ["android", "navigation", "jetpack"]
categories: ["Android"]
---

# Introduction

When you start a new job in an established company, you expect what is already in production to be working well, because it's in prod, right? Well, that's not always the case, unfortunately. 

This is the story on how that app already in production was not setup correctly from the ground up and the lessons learned along the way to fix it!

# The Wrong Setup

This app is a little particular because it's a SaaS product, where there's a signup/login part, and only after logging in successfully, the user can land on the main screen, a dashboard.

![login-dashboard-flow](/assets/img/login-dashboard-flow.drawio.png)

As there are always two steps, it was decided to implement this flow as two separate consecutive activities.

![login-dashboard-flow](/assets/img/two-activities-flow.drawio.png)

There's a **LoginActivity** which redirects the user to a **MainActivity** if the login was successful, with the help of a **LoginManager** (Singleton) in the app, which keeps the current login state and some info like the username, in-memory. A screen like **DashboardFragment** then presents that username in its UI.

This setup gave everyone a false feeling of security as **DashboardFragment** is *assumed* to show up only after logging in because it is only contained by the second activity in the flow, which shows up **only** if the user is logged-in.

Then, bugs from **production** started to pour in telling us that sometimes the username is empty! **How come?** The elephant in the room was obviously **[System-initiated Process Death](https://galex.dev/posts/process-death-is-the-rule-not-the-exception/)**. 

After the user has logged-in, used the dashboard for a while, then switched to another app, and our favorite Android mechanism of [Process Death](https://galex.dev/posts/process-death-is-the-rule-not-the-exception/) occurred, the Android OS brought the user back to where they were, but the process was recreated, **LoginManager** contains no data and **nothing** was done in **DashBoardFragment** to recover from this situation!

# The Meh Patch

One possible fix to this is to check via the **LoginManager** that the user is actually logged-in and if not, redirect the user back to the **LoginActivity**.

That could work if there was only one screen/fragment in the **MainActivity**. There were multiple long flows in there that open from the Dashboard, so the user would completely lose those flows if we were to patch it this way. Meh.

# The Right Fix

On Android, **Every Screen Is An Entry Point**. As we know, Android restores the whole activities stack and it each activity, the fragments stack.

In this use case, Android will restore the **DashboardFragment** (inside **MainActivity**) and it should **ITSELF** navigate the user to the **LoginFragment** (inside **MainActivity** as well) on top of itself!

Always be conscious about [System-initiated Process Death](https://galex.dev/posts/process-death-is-the-rule-not-the-exception/)!
