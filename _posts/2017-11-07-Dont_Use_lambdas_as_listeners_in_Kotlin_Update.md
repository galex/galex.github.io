---
layout: post
title: Don't use lambdas as listeners in Kotlin - Update
category:
---

Thank you to everyone of you who read the [original blogpost](http://galex.co.il/2017/11/04/Dont_Use_lambads_as_listeners_in_Kotlin.html) and even more to those who [commented on it](http://disq.us/t/2vkteux) or on [Reddit](https://www.reddit.com/r/androiddev/comments/7asv5g/dont_use_lambdas_as_listeners_in_kotlin/?ref=share&ref_source=link)!

## The TLDR; Update

As mentioned in the comments we actually can declare a lambda / function literal with an adapter function (the OnAudioFocusChangeListener interface):

{% highlight kotlin %}
onAudioFocusChange = AudioManager.OnAudioFocusChangeListener { focusChange: Int ->
        Log.d(TAG, "In onAudioFocusChange focus changed to = $focusChange")
        // do stuff
}

{% endhighlight %}
This lambda is then used to define the body of the class that will be generated for Java via the [SAM Conversion](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions), as in our case, the we use API is written in Java so Java interop is part of the game.

## The Longer Update

### Is **Lateinit** guilty or not?

Some argued that the problem is the declaration of the listener using the **lateinit** keyword. To check if **lateinit** is indeed guilty, let's reproduce the same lambda with and without **lateinit** and see how it is used.

To remind us what we're talking about, here's the code of the two lambdas in Kotlin:
{% highlight kotlin %}
// with lateinit
private lateinit var onAudioFocusChangeListener1: (focusChange: Int) -> Unit

// without lateinit
private val onAudioFocusChangeListener2: (focusChange: Int) -> Unit = { focusChange: Int ->
    Log.d(TAG, "In onAudioFocusChangeListener2 focus changed to = $focusChange")
    // do some stuff
}

// in onCreate()
onAudioFocusChangeListener1 = { focusChange: Int ->
    Log.d(TAG, "In onAudioFocusChangeListener1 focus changed to = $focusChange")
    // do some stuff
}

{% endhighlight %}

### With **lateinit** (onAudioFocusChangeListener1)

{% highlight java %}
// Declaration
private Function1<? super Integer, Unit> onAudioFocusChangeListener1;

// in onCreate()
this.onAudioFocusChangeListener1 = MainActivity$onCreate$1.INSTANCE;

// Class implementation
final class MainActivity$onCreate$1 extends Lambda implements Function1<Integer, Unit> {
    public static final MainActivity$onCreate$1 INSTANCE = new MainActivity$onCreate$1();

    MainActivity$onCreate$1() {
        super(1);
    }

    public final void invoke(int focusChange) {
        Log.d(MainActivity.Companion.getTAG(), "In onAudioFocusChangeListener1 focus changed to = " + focusChange);
    }
}

// In onCreate(), a button uses a SAM converted lambda to call the AudioManager API
Function1 listener = this.onAudioFocusChangeListener1;
((Button) findViewById(C0220R.id.obtain)).setOnClickListener(new MainActivity$onCreate$2(this, listener));

// Inside MainActivity$onCreate$2 the call to the AudioManager API
if (function1 != null) {
    mainActivityKt$sam$OnAudioFocusChangeListener$4186f324 = new MainActivityKt$sam$OnAudioFocusChangeListener$4186f324(function1);
} else {
    Object obj = function1;
}
Log.d(MainActivity.Companion.getTAG(), "granted = " + (access$getAudioManager$p.requestAudioFocus((OnAudioFocusChangeListener) mainActivityKt$sam$OnAudioFocusChangeListener$4186f324, 3, 1) == 1));
{% endhighlight %}

Our lambda / function literal is wrapped inside a class implementing the interface ([SAM Conversion](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions)), which is the whole issue here.

### Without **lateinit** (onAudioFocusChangeListener2)

{% highlight java %}
// Declaration of the lambda
private final Function1<Integer, Unit> onAudioFocusChangeListener2 = MainActivity$onAudioFocusChangeListener2$1.INSTANCE;

// Class implementation
final class MainActivity$onAudioFocusChangeListener2$1 extends Lambda implements Function1<Integer, Unit> {
    public static final MainActivity$onAudioFocusChangeListener2$1 INSTANCE = new MainActivity$onAudioFocusChangeListener2$1();

    MainActivity$onAudioFocusChangeListener2$1() {
        super(1);
    }

    public final void invoke(int focusChange) {
        Log.d(MainActivity.Companion.getTAG(), "In onAudioFocusChangeListener1 focus changed to = " + focusChange);
    }
}

// In onCreate(), a button uses a SAM converted lambda to call the AudioManager API
Function1 listener = this.onAudioFocusChangeListener2;
((Button) findViewById(C0220R.id.obtain)).setOnClickListener(new MainActivity$onCreate$2(this, listener));

// Inside MainActivity$onCreate$2 the call to the AudioManager API
if (function1 != null) {
    mainActivityKt$sam$OnAudioFocusChangeListener$4186f324 = new MainActivityKt$sam$OnAudioFocusChangeListener$4186f324(function1);
} else {
    Object obj = function1;
}
Log.d(MainActivity.Companion.getTAG(), "granted = " + (access$getAudioManager$p.requestAudioFocus((OnAudioFocusChangeListener) mainActivityKt$sam$OnAudioFocusChangeListener$4186f324, 3, 1) == 1));
{% endhighlight %}

Same issue without **lateinit**, so we hold no charges against it.

### The preferable way

To fix the issue, I recommended using an anonymous inner class:

{% highlight kotlin %}
private val onAudioFocusChangeListener3: AudioManager.OnAudioFocusChangeListener = object : AudioManager.OnAudioFocusChangeListener {
    override fun onAudioFocusChange(focusChange: Int) {
        Log.d(TAG, "In onAudioFocusChangeListener2 focus changed to = $focusChange")
        // do some stuff
    }
}
{% endhighlight %}
Which translates to the following in Java:
{% highlight java %}

// declaration
private final OnAudioFocusChangeListener onAudioFocusChangeListener3 = new MainActivity$onAudioFocusChangeListener3$1();

// class definition
public final class MainActivity$onAudioFocusChangeListener3$1 implements OnAudioFocusChangeListener {
    MainActivity$onAudioFocusChangeListener3$1() {
    }

    public void onAudioFocusChange(int focusChange) {
        Log.d(MainActivity.Companion.getTAG(), "In onAudioFocusChangeListener2 focus changed to = " + focusChange);
    }
}

// In onCreate(), a button uses a SAM converted lambda to call the AudioManager API
OnAudioFocusChangeListener listener = this.onAudioFocusChangeListener3;
((Button) findViewById(C0220R.id.obtain)).setOnClickListener(new MainActivity$onCreate$2(this, listener));

// Inside MainActivity$onCreate$2 the call to the AudioManager API
Log.d(MainActivity.Companion.getTAG(), "Calling AudioManager.requestAudioFocus()");
int focusRequest = MainActivity.access$getAudioManager$p(this.this$0).requestAudioFocus(this.$listener, 3, 1);
{% endhighlight %}

The anonymous class implements the expected interface and the same instance is used (the compiler does not need to use a  [SAM Conversion](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions) as no lambdas are involved). Neat!

### The best way

The most concise way is to actually declare the lambda and use what the [documentation](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions) calls an adapter function:

{% highlight kotlin %}
private val onAudioFocusChangeListener4 = AudioManager.OnAudioFocusChangeListener { focusChange: Int ->
    Log.d(TAG, "In onAudioFocusChangeListener3 focus changed to = $focusChange")
    // do some stuff
}
{% endhighlight %}

This indicates to the compiler that this is the type to use when doing a [SAM Conversion](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions) on it. It translates to the following in Java:
{% highlight java %}
// declaration
private final OnAudioFocusChangeListener onAudioFocusChangeListener4 = MainActivity$onAudioFocusChangeListener4$1.INSTANCE;

// Class definition
final class MainActivity$onAudioFocusChangeListener4$1 implements OnAudioFocusChangeListener {
    public static final MainActivity$onAudioFocusChangeListener4$1 INSTANCE = new MainActivity$onAudioFocusChangeListener4$1();

    MainActivity$onAudioFocusChangeListener4$1() {
    }

    public final void onAudioFocusChange(int focusChange) {
        Log.d(MainActivity.Companion.getTAG(), "In onAudioFocusChangeListener3 focus changed to = " + focusChange);
    }
}

// In onCreate(), a button uses a SAM converted lambda to call the AudioManager API
OnAudioFocusChangeListener listener = this.onAudioFocusChangeListener4;
((Button) findViewById(C0220R.id.obtain)).setOnClickListener(new MainActivity$onCreate$2(this, listener));

// Inside MainActivity$onCreate$2 the call to the AudioManager API
Log.d(MainActivity.Companion.getTAG(), "Calling AudioManager.requestAudioFocus()");
int focusRequest = MainActivity.access$getAudioManager$p(this.this$0).requestAudioFocus(this.$listener, 3, 1);
{% endhighlight %}

## Conclusion

As perfectly put by **Roman Dawydkin** on [Slack](http://slack.kotlinlang.org/):

> You *can* use lambda as a listener if you create only one instance of it

It's not an issue if the lambda is used in a functional way or as a callback. The issue occurs only when used as listeners with an API **written in Java** which expects the same object reference in an observer design pattern. If the API would have been written in Kotlin there would have been no [SAM Conversion](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions) involved thus no issue. We'll get there one day!

I hope the subject is now crystal clear for everyone!

I'd like to thank Rhaquel Gherschon for for proofreading and [Christophe Beyls](https://twitter.com/BladeCoder) for giving me feedback on this article!

Cheers!
