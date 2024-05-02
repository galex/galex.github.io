---
layout: post
title:  "State Management in Activities"
tags: ["android", "process death", "solution", "state", "saveState", "restoreState", "Activity"]
categories: ["Android", "State Management"]
mermaid: true
comments: true
---

![People holding phones in a party](/assets/img/header-beach.png)

Activities are the Entry Point to an Android App. 
They are the first thing the user sees when they open the app. 
They are also the first thing the user sees when they return to the app after a long time. 

When the user leaves the app, the Android OS can decide to terminate the app's process to free up resources. This is called **Process Death**. 
When the user returns to the app, the Android OS can decide to revive the app. This is called **Process Revival**.
When the Android OS revives the app, it will try to restore the state of the app as much as possible. This is called **State Restoration**.


### State Management in Activities

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

Let's see how it behaves when the screen orientation changes:

{% include video.html path="/assets/vids/activity-enter-name-value-lost.mp4" %}

When the configuration change or Process Death occurs, the entered value disappears! 

Let's fix this by saving and restoring the `name` variable.

Activities have one main method to help us **save** their state:

- `onSaveInstanceState(Bundle outState)`: This method is called before the Activity is destroyed. We can use it to save the state of the Activity.

And two main methods to help us restore their state:

- `onCreate(Bundle savedInstanceState)`: This method is called when the Activity is created. We can use it to restore the state of the Activity.
- `onRestoreInstanceState(Bundle savedInstanceState)`: This method is called after the Activity is recreated. It is called after `onStart()` and before `onResume()`.

> ℹ️ The `onRestoreInstanceState` method is not called if the Activity is created for the first time. It is only called when the Activity is recreated after being destroyed.
> Using `onRestoreInstanceState()` is a matter of use cases. Usually, we'll use `onCreate()` to restore the state of the Activity.

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

**As soon as a View has an ID**, Android will save its state and restore it **automatically**.

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
We can deduct that `saveHierarchyState()` works by building a Bundle that contains the state of all the Views in the Activity by building a tree of Bundles. Each Bundle represents the state of a View and is indexed by the View's ID.
When the Activity is recreated, `restoreHierarchyState()` is called with the Bundle that contains the state of all the Views. It will restore the state of each View by their IDs.

Now that we know this, let's see what how this happens on the other side of the mirror, inside [Views](#views).

### Views

Let's create a very simple Bottom Bar View that will contain a few buttons:

```kotlin
class BottomBarView(context: Context, attrs: AttributeSet) : LinearLayout(context, attrs) {

    private var icon1: ImageView
    private var icon2: ImageView
    private var icon3: ImageView
    private var selectedIconId: Int = R.id.home

    init {
        orientation = HORIZONTAL
        icon1 = ImageView(context).apply { id = R.id.home; setImageResource(R.drawable.ic_home) }.also { addView(it) }
        addView(Space(context).apply { weightSum = 1F })
        icon2 = ImageView(context).apply { id = R.id.account; setImageResource(R.drawable.ic_account) }.also { addView(it) }
        addView(Space(context).apply { weightSum = 1F })
        icon3 = ImageView(context).apply { id = R.id.settings; setImageResource(R.drawable.ic_settings) }.also { addView(it) }
    }

// Other methods to handle user interaction and update the state of the icons...

```

### Fragments

### ViewModels

### Official documentation

### Conclusion
