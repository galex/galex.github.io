---
layout: post
title:  "Unveiling AndroidFragment: Migrating Navigation from Fragments to Compose"
tags: ["android", "navigation", "compose", "fragments"]
categories: ["Android", "Navigation"]
mermaid: true
comments: true
---

![Formula 1 on a highway](/assets/img/header-formula-1.png)

Since [Jetpack Compose](https://developer.android.com/develop/ui/compose) came out in 2021, many Android developers have been migrating their apps to this new UI toolkit.
However, one of the biggest challenges is migrating the navigation system from the old Fragment-based one to the new Compose-based one.

The main plan to migrate from Fragments to Compose was to:
1. Use Compose as UI inside Fragments
2. Build all the infrastructure built in Fragments in Compose (Analytics, etc.)
3. Replace the Fragment-based navigation with Compose-based navigation

For little projects this is easily done in a few days but for big companies, this is a **huge task** that would probably not see the light for many years!

So instead of migrating all fragments one by one, **is there a way to simply use Fragments inside Compose?**

### AndroidViewBinding

It is indeed already possible using `AndroidViewBinding`, but only if our Fragment has **no arguments** because it is taking advantage of `FragmentContainerView`, in which the Fragment class name is declared in XML and instantiated for us under the hood:

```kotlin
@Composable
fun DashboardScreen() {
    val context = LocalContext.current
    val binding = FragmentDashboardBinding.inflate(LayoutInflater.from(context))
    AndroidViewBinding(FragmentDashboardBinding::inflate) {
        // Do something on the binding if necessary
    }
}
```
Where `dashboard_fragment.xml` layout looks like this:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:layout_width="match_parent"
              android:layout_height="match_parent"
              android:orientation="vertical">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fragmentContainer"
        android:name="dev.galex.compose.app.DashboardFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```

### AndroidFragment

`AndroidFragment` is a new `@Composable` that allows us to use Fragments directly inside Compose.

Inside our own `@Composable`, we can use `AndroidFragment` like this:

```kotlin
@Composable
fun ItemScreen(
    modifier: Modifier = Modifier,
    itemId: String,
) {
    AndroidFragment<ItemFragment>(
        arguments = bundleOf(ItemFragment.ARG_ID to "some-id"),
        modifier = modifier.fillMaxSize()
    )
}
```

`AndroidFragment` is available since version `1.8.0-alpha01` of the `androidx.fragment:fragment-compose` library, by adding the following to our `libs.version.toml`:

```toml
[versions]
fragment = "1.8.0-alpha02"

[libraries]
androidx-fragment-compose = { group = "androidx.fragment", name = "fragment-compose", version.ref = "fragment" }
```
And adding this dependency to our `build.gradle.kts` app module:

```kotlin
dependencies {
    implementation(libs.androidx.fragment.compose)
}
```

> ⚠️ This is still in alpha, so I'd wait for `1.8.0` to be stable before using it in production

Documentation is available [here](https://developer.android.com/jetpack/androidx/releases/fragment#1.8.0-alpha01).

## Conclusion

I am very excited for `AndroidFragment` because we're not dependent anymore on rewriting all Fragments to Composables first before being able to switch to a Compose-based Navigation System. 
This will allow us to migrate to Compose in a more gradual way, which is a huge win!

Feel free to comment below if you have any questions!

By the way, I'm also on [Twitter](https://twitter.com/galex) and [LinkedIn](https://www.linkedin.com/in/agherschon/), so feel free to connect there too!

Stay tuned for more Android State Management deep dives! 🚀
