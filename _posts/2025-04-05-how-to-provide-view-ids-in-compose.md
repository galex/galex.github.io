---
layout: post
title:  "How to provide View IDs in Jetpack Compose"
tags: ["android", "jetpack", "compose", "automation", "qa", "maestro", "appium"]
categories: ["Android", "Compose"]
mermaid: true
comments: true
---
![Waterfall flowing into the Sea](/assets/img/header-waterfall.png)

## Introduction

Automation Frameworks like [Appium Inspector](https://github.com/appium/appium-inspector) or [Maestro Studio](https://docs.maestro.dev/getting-started/maestro-studio) need to find views ids in our mobile app to be able to interact with its views.

But in the new world of Jetpack Compose, we don't have XML files anymore, so we don't have View Ids anymore, so how do we provide those IDs to the automation framework?

## Definition of Done

Before we start, let's define what we want to achieve:
- We want to be able to provide View IDs to any of our Composables
- We want the IDs to be 100% unique, meaning we don't have to worry about the same id existing twice in a list, aka the most common issue with RecyclerView
- We want to relate the ids to the "context" of the screen

I call this whole thing `AutomationContext` because we're using it to provide the context (and ids) of the screen to an Automation framework.

## Code Project

Let's have a `ContactsScreen` that shows a list of contacts:

```kotlin
@Composable
fun ContactsScreen(contacts: List<Contact>) {
    LazyColumn(
        modifier = Modifier
            .fillMaxSize()
    ) {
        items(contacts) { contact ->
            ContactItem(contact)
        }
    }
}
```
And a `ContactItem` that shows the contact name and phone number:

```kotlin
@Composable
fun ContactItem(contact: Contact) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .padding(8.dp),
    ) {
        Row(
            modifier = Modifier.padding(16.dp),
            verticalAlignment = Alignment.CenterVertically,
        ) {
            Icon(
                imageVector = Icons.Filled.Person,
                contentDescription = "Person Icon",
                tint = MaterialTheme.colorScheme.onSurfaceVariant,
                modifier = Modifier
                    .size(50.dp)
                    .clip(CircleShape),
            )
            Spacer(modifier = Modifier.width(16.dp))
            Column {
                Text(text = contact.name, fontWeight = FontWeight.Bold)
                Spacer(modifier = Modifier.height(4.dp))
                Text(text = contact.phoneNumber)
            }
        }
    }
}
```
Now let's look at what Maestro Studio sees when we run the app and launch `maestro studio`:

```
 ~  maestro studio
Running on 19181FDF600E9W

╭────────────────────────────────────────────────────────╮
│                                                        │
│   Maestro Studio is running at http://localhost:9999   │
│                                                        │
╰────────────────────────────────────────────────────────╯


Tip: Maestro Studio can now run simultaneously alongside other Maestro CLI commands!

Navigate to http://localhost:9999 in your browser to open Maestro Studio. Ctrl-C to exit.
```
This opens automatically the browser on that url (`http://localhost:9999`) and we can see the following:

![Maestro Studio](/assets/img/maestro-studio-at-start.png)

We see some view ids from the Android system like the clock or the battery, and we do see the text of our views, but we don't see any view ids for our items, as expected.

## First Approach: Composables

The basic idea is that we'll use a `CompositionLocal` to provide a `context` (String) that will be chained to any previously provided `context` (String) in the composition tree, then use that to set resource ids via `Modifier.testTag()`.

### Step 1: Create the CompositionLocal

We'll create a `CompositionLocal` that will hold the context (String):

```kotlin
private val LocalAutomationContext = compositionLocalOf { "" }
```

### Step 2: Create the AutomationContext Composable

```kotlin
@Composable
fun AutomationContext(context: String, content: @Composable () -> Unit) {
    val prev = LocalAutomationContext.current
    val chained = if (prev.isEmpty()) context else "${prev}_${context}"
    CompositionLocalProvider(LocalAutomationContext provides chained, content = content)
}
```
And we'll also create a helper function called `AutomationIndex` based on `AutomationContext` to provide the index of the item in the list:

```kotlin
@Composable
fun AutomationIndex(index: Int, content: @Composable () -> Unit) {
    AutomationContext("index_$index", content)
}
```

### Step 3: Create the AutomationId Composable

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun AutomationId(id: String, content: @Composable () -> Unit) {
    val packagePrefix = LocalContext.current.packageName
    val prev = LocalAutomationContext.current
    val newContext = if (prev.isEmpty()) id else "${prev}_${id}"
    val newId = "$packagePrefix:id/$newContext"
    CompositionLocalProvider(LocalAutomationContext provides newContext) {
        Box(
            modifier = Modifier
                .semantics { testTagsAsResourceId = true }
                .testTag(newId)
        ) {
            content()
        }
    }
}
```
### Step 4: Use those new Composables in our ContactsScreen

We'll wrap the content of the screen with `AutomationContext`, use `AutomationIndex` to provide the index of the item in the list, and use `AutomationId` to provide the id of the card, icon, name and phone number:

```kotlin
@Composable
fun ContactsScreen(contacts: List<Contact>) {
    AutomationContext("contacts") { // Set the context for the screen
        LazyColumn(
            modifier = Modifier
                .fillMaxSize()
        ) {
            itemsIndexed(contacts) { index, contact ->
                AutomationIndex(index) { // Set the index for each item
                    ContactItem(contact)
                }
            }
        }
    }
}

@Composable
fun ContactItem(contact: Contact) {
    AutomationId("card") { // Set the ID for the card as a container
        Card(
            modifier = Modifier
                .fillMaxWidth()
                .padding(8.dp),
        ) {
            Row(
                modifier = Modifier.padding(16.dp),
                verticalAlignment = Alignment.CenterVertically,
            ) {
                AutomationId("icon") { // Set the ID for the icon
                    Icon(
                        imageVector = Icons.Filled.Person,
                        contentDescription = "Person Icon",
                        tint = MaterialTheme.colorScheme.onSurfaceVariant,
                        modifier = Modifier
                            .size(50.dp)
                            .clip(CircleShape),
                    )
                }
                Spacer(modifier = Modifier.width(16.dp))
                Column {
                    AutomationId("name") { // Set the ID for the name
                        Text(text = contact.name, fontWeight = FontWeight.Bold)
                    }
                    Spacer(modifier = Modifier.height(4.dp))
                    AutomationId("phone") { // Set the ID for the phone number
                        Text(text = contact.phoneNumber)
                    }
                }
            }
        }
    }
}
```

### First approach: Result 

Now we can see that Maestro Studio is finally able to see the ids of our views:

![Maestro Studio](/assets/img/maestro-studio-with-ids.png)

This first approach has a big issue which is that we have to wrap every single Composable with `AutomationContext`, `AutomationInex` and `AutomationId`, which is far from ideal, to say the least! 🤮

## Second Approach: Modifiers

We can write this differently using [ModifierLocal](https://developer.android.com/reference/kotlin/androidx/compose/ui/modifier/ModifierLocal) which allows us to pass information through the *Modifier tree*, which is exactly what we need!

To use `ModifierLocal`, we need to:
- Consume the previously provided context via `Modifier.modifierLocalConsumer {}`
- Provide the new context via `Modifier.modifierLocalProvider() {}`

### Step 1: Create the ModifierLocal

We'll create a `ModifierLocal` that will hold the context (String):

```kotlin
private val ModifierLocalAutomationContext = modifierLocalOf { "" }
```

### Step 2: Create the AutomationContext and AutomationIndex Modifiers

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun Modifier.automationContext(context: String): Modifier {
    var prev by remember { mutableStateOf<String?>(null) }
    return this then Modifier
        .modifierLocalConsumer { prev = ModifierLocalAutomationContext.current }
        .modifierLocalProvider(ModifierLocalAutomationContext) {
            if (prev.isNullOrEmpty()) context else "${prev}_${context}"
        }
}

@Composable
fun Modifier.automationIndex(index: Int): Modifier = automationContext("index_$index")
```

### Step 3: Create the AutomationId Modifier

```kotlin
@Composable
fun Modifier.automationId(id: String): Modifier {
    val packagePrefix = LocalContext.current.packageName
    var prev by remember { mutableStateOf<String?>(null) }
    return this then Modifier
        .modifierLocalConsumer { prev = ModifierLocalAutomationContext.current }
        .semantics { testTagsAsResourceId = true }
        .testTag(if (prev.isNullOrEmpty()) "$packagePrefix:id/$id" else "$packagePrefix:id/${prev}_${id}")
}
```

### Step 4: Use those new Modifiers in our ContactsScreen

Now that we don't need to wrap anything, we'll simply use the new modifiers in our Composables:

```kotlin
@Composable
fun ContactsScreen(contacts: List<Contact>) {
    LazyColumn(
        modifier = Modifier
            .fillMaxSize()
            .automationContext("contacts") // Set the context for the screen
    ) {
        itemsIndexed(contacts) { index, contact ->
            ContactItem(
                contact = contact,
                modifier = Modifier
                    .automationIndex(index) // Set the index for the item
            )
        }
    }
}

@Composable
fun ContactItem(
    contact: Contact,
    modifier: Modifier = Modifier,
) {
    Card(
        modifier = modifier
            .fillMaxWidth()
            .padding(8.dp)
            .automationId("card"), // Set the ID for the card
    ) {
        Row(
            modifier = Modifier.padding(16.dp),
            verticalAlignment = Alignment.CenterVertically,
        ) {
            Icon(
                imageVector = Icons.Filled.Person,
                contentDescription = "Person Icon",
                tint = MaterialTheme.colorScheme.onSurfaceVariant,
                modifier = Modifier
                    .size(50.dp)
                    .clip(CircleShape)
                    .automationId("icon") // Set the ID for the icon,
            )

            Spacer(modifier = Modifier.width(16.dp))
            Column {
                Text(
                    text = contact.name,
                    fontWeight = FontWeight.Bold,
                    modifier = Modifier
                        .automationId("name"), // Set the ID for the name
                )
                Spacer(modifier = Modifier.height(4.dp))
                Text(
                    text = contact.phoneNumber,
                    modifier = Modifier
                        .automationId("phone"), // Set the ID for the phone number
                )
            }
        }
    }
}
```

### Second approach: Result

Exactly the same as the first approach in terms of view ids seen by Automation Frameworks, let's filter this time and look at the ids only on index `4`:

![Maestro Studio](/assets/img/maestro-studio-id-row-4.png)

## Conclusion

The second approach is much cleaner and easier to use, and we got exactly what we wanted: unique view ids, per row, that are related to the context of the screen! ❤️

I've prepared you a Gist with all the relevant code so you can copy/paste it in your project and start using it right away: [Gist](https://gist.github.com/galex/28d5ad2325ebbc633d7f4e4ee98c6ee1) 💪

Let me know what you think or if you have any questions in the comments! 📝

Happy composing! 🧪



