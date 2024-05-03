---
layout: post
title:  "State Management in Activities"
tags: ["android", "process death", "solution", "state", "saveState", "restoreState", "Activity"]
categories: ["Android", "State Management"]
mermaid: true
comments: true
---

![Standing in front of a big main entrance](/assets/img/header-big-entrance.png)

Activities are the Entry Point to an Android App. 
They are the first thing the user sees when they open the app. 
They are also the first thing the user sees when they return to the app after a long time. 

When the user leaves the app, the Android OS can decide to terminate the app's process to free up resources. This is called **Process Death**. 
When the user returns to the app, the Android OS can decide to revive the app.
When the Android OS revives the app after the user reopens it, it will try to restore the state of the app as much as possible. This is called **State Restoration**.


## No State Management

We'll create an Activity where the layout is created dynamically to raise a few interesting points.

```kotlin
class EnterNameActivity : AppCompatActivity() {

    private var name: String? = null
    private lateinit var editTextName: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // editText created dynamically
        editTextName = EditText(this).apply {
            hint = "Enter name"
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
        }

        val layout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            addView(editTextName)
        }

        setContentView(layout)

        editTextName.addTextChangedListener { text ->
            name = text.toString()
        }
    }
}
```

We created an `EditText` dynamically and added a `TextWatcher` to it to save the entered value in the `name` variable. 
This `EditText` is created dynamically, and it doesn't have an ID.
What happens when a View doesn't have an ID?
Let's see how it behaves when the screen orientation changes:

{% include video.html path="/assets/vids/activity-enter-name-value-lost.mp4" %}

When the configuration change or Process Death occurs, the entered value disappears! 

Let's fix this by saving and restoring the `name` variable.

Activities have one main method to help us **save** their state:

- `onSaveInstanceState(Bundle outState)`: This method is called before the Activity is destroyed, so we can use it to save the state of the Activity

And two main methods to help us restore their state:

- `onCreate(Bundle savedInstanceState)`: This method is called when the Activity is created, so then we can use it to restore the state of the Activity
- `onRestoreInstanceState(Bundle savedInstanceState)`: This method is called after the Activity is recreated. It is called after `onStart()` and before `onResume()`

> ℹ️ The `onRestoreInstanceState` method is not called if the Activity is created for the first time. It is only called when the Activity is recreated after being destroyed.
> Using `onRestoreInstanceState()` is a matter of use cases. Usually, we'll use `onCreate()` to restore the state of the Activity.

## Manual State Management

Let's modify the `EnterNameActivity` to save and restore the `name` variable:

```kotlin
class EnterNameActivity : AppCompatActivity() {

    private var name: String? = null
    private lateinit var editTextName: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // editText created dynamically
        editTextName = EditText(this).apply {
            hint = "Enter name"
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
        }

        val layout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            addView(editTextName)
        }

        setContentView(layout)

        editTextName.addTextChangedListener { text ->
            name = text.toString()
        }

        // Restore the name value from savedInstanceState
        name = savedInstanceState?.getString("name")
        // Set the restored name value to the EditText
        name?.let { editTextName.setText(it) }
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putString("name", name)
    }
}
```

Here we used the `onSaveInstanceState` method to save the `name` variable and the `onCreate` method to restore it. 
Let's see how it behaves now when the screen orientation changes, and also when Process Death occurs:

{% include video.html path="/assets/vids/activity-enter-name-value-saved.mp4" %}

We're all good! We created manually a View and managed its state at the Activity level. 

But if we take a second to imagine having to do that for each View, I am pretty sure Android wouldn't have been adopted as much as it is today!
**When a View has an ID**, Android will save its state and restore it **automatically**.

## Automatic View State Management

Let's declare an ID for the `EditText`:

```xml
<resources>
    <id name="name_edit_text" />
</resources>
```

And let's set it in the EditText:

```kotlin
class EnterNameActivity : AppCompatActivity() {

    private var name: String? = null
    private lateinit var editTextName: EditText

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // editText created dynamically
        editTextName = EditText(this).apply {
            hint = "Enter name"
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
            // Setting the ID on the created View  
            id = R.id.name_edit_text
        }

        val layout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            addView(editTextName)
        }

        setContentView(layout)

        editTextName.addTextChangedListener { text ->
            name = text.toString()
        }
    }
}
```
We removed the manual saving and restoring of the `name` variable and added an ID that we created by declaring a resource id. Let's reproduce again Configuration Change and Process Death:

{% include video.html path="/assets/vids/activity-enter-name-value-saved-automatically.mp4" %}

When a View has a proper id that will not change at runtime like a resource id, Android will save and restore its state automatically.
Android does this with the call to `saveHierarchyState()` on the `Window` object in the `onSaveInstanceState` method of the `android.app.Activity` class:
```kotlin
// Saving View Hierarchy In android.app.Activity class
protected fun onSaveInstanceState(@NonNull outState: Bundle) {
    outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState())
    // ...
}

// Restoring View Hierachy in android.app.Activity class
protected fun onRestoreInstanceState(@NonNull savedInstanceState: Bundle) {
    if (mWindow != null) {
        val windowState: Bundle = savedInstanceState.getBundle(WINDOW_HIERARCHY_TAG)
        if (windowState != null) {
            mWindow.restoreHierarchyState(windowState)
        }
    }
}
````
We can deduct that `saveHierarchyState()` works by building a Bundle that contains the state of all the Views in the Activity by building a map of Bundles. Each Bundle represents the state of a View and is indexed by the View's ID.
When the Activity is recreated, `restoreHierarchyState()` is called with the Bundle that contains the state of all the Views. It will restore the state of each View by their IDs.

## Conclusion

We learned how to save and restore the state of an Activity manually. 
We also learned how Android saves and restores the state of Views automatically when they have a proper ID.

Feel free to comment below if you have any questions!

By the way, I'm also on [Twitter](https://twitter.com/galex) and [LinkedIn](https://www.linkedin.com/in/agherschon/), so feel free to connect there too!

Stay tuned for more Android State Management deep dives! 🚀
