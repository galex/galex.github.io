---
layout: post
title: Don't use lambdas as listeners in Kotlin
category:
---
I had this issue on the first Android app I'm writing entirely in Kotlin and it drove me **crazy**!

## The setup

I'm implementing [Audio Focus](https://developer.android.com/guide/topics/media-apps/audio-focus.html) in a podcast app. When a user wants to play an episode of the podcast, we need to **request the audio focus** passing by an implementation of [OnAudioChangeListener](https://developer.android.com/reference/android/media/AudioManager.OnAudioFocusChangeListener.html) (because we could lose the audio focus while playing if the user uses another app which also requests audio focus):

{% highlight kotlin %}
private fun requestAudioFocus(): Boolean {
    Log.d(TAG, "requestAudioFocus() called")
    val focusRequest: Int = audioManager.requestAudioFocus(onAudioFocusChange,
        AudioManager.STREAM_MUSIC,
        AudioManager.AUDIOFOCUS_GAIN)
    return focusRequest == AudioManager.AUDIOFOCUS_REQUEST_GRANTED
    }
{% endhighlight %}

In this listener we want to react to different states:

{% highlight kotlin %}
when (focusChange) {
    AudioManager.AUDIOFOCUS_GAIN -> TODO("resume playing")
    AudioManager.AUDIOFOCUS_LOSS -> TODO("abandon focus and stop playing")
    AudioManager.AUDIOFOCUS_LOSS_TRANSIENT -> TODO("pause but keep focus")
}
{% endhighlight %}

When the episode is done playing or the user actively pauses the episode he is listening to, we need to **abandon the audio focus**:

{% highlight kotlin %}
private fun abandonAudioFocus(): Boolean {
    Log.d(TAG, "abandonAudioFocus() called")
    val focusRequest: Int = audioManager.abandonAudioFocus(onAudioFocusChange)
    return focusRequest == AudioManager.AUDIOFOCUS_REQUEST_GRANTED
}
{% endhighlight %}

## The Path to Madness

As avid of new stuff as I am I decided to implement the listener, **onAudioFocusChange**, as a lambda function. I don't remember if it was suggested by the IntelliJ IDE or not, but anyway it was declared as the following:

{% highlight kotlin %}
private lateinit var onAudioFocusChange: (focusChange: Int) -> Unit
{% endhighlight %}

In onCreate() this variable is assigned a lambda function:

{% highlight kotlin %}
onAudioFocusChange = { focusChange: Int ->
    Log.d(TAG, "In onAudioFocusChange focus changed to = $focusChange")
    when (focusChange) {
        AudioManager.AUDIOFOCUS_GAIN -> TODO("resume playing")
        AudioManager.AUDIOFOCUS_LOSS -> TODO("abandon focus and stop playing")
        AudioManager.AUDIOFOCUS_LOSS_TRANSIENT -> TODO("pause but keep focus")
    }
}
{% endhighlight %}

All worked fine as we can request the audio focus which would pause other apps (like Spotify) and play our episode.

Abandoning the audio focus seemed to work too as we get [**AUDIOFOCUS_REQUEST_GRANTED**](https://developer.android.com/reference/android/media/AudioManager.html#AUDIOFOCUS_REQUEST_GRANTED) as a result when calling **abandonAudioFocus** on **AudioManager**.

{% highlight log %}
11-04 16:08:14.610 D/MainActivity: requestAudioFocus() called
11-04 16:08:14.618 D/AudioManager: requestAudioFocus status : 1
11-04 16:08:14.619 D/MainActivity: granted = true
11-04 16:09:34.519 D/MainActivity: abandonAudioFocus() called
11-04 16:09:34.521 D/MainActivity: granted = true
{% endhighlight %}

But as soon as we want to request again the audio focus we are notified through the listener we immediately lost the focus by receiving [**AUDIO_FOCUS_LOSS**](https://developer.android.com/reference/android/media/AudioManager.html#AUDIOFOCUS_LOSS):

{% highlight log %}
11-04 16:17:38.307 D/MainActivity: requestAudioFocus() called
11-04 16:17:38.312 D/AudioManager: requestAudioFocus status : 1
11-04 16:17:38.312 D/MainActivity: granted = true
11-04 16:17:38.321 D/AudioManager: AudioManager dispatching onAudioFocusChange(-1)
   // for MainActivityKt$sam$OnAudioFocusChangeListener$4186f324$828aa1f
11-04 16:17:38.322 D/MainActivity: In onAudioFocusChange focus changed to = -1
{% endhighlight %}

Why are we notified of losing the audio focus after requesting it?

What the hell is going on?

## Behind the scene

The best tool to help us understand the issue is the **Kotlin Bytecode** viewer!

![Kotlin Bytecode viewer]({{ site.url }}/assets/dont-use-lambdas-as-listeners/kotlin-bytecode.png){: .center-image }

![Decompile button]({{ site.url }}/assets/dont-use-lambdas-as-listeners/decompile.png){: .center-image }

Let's have a look at what is assigned to our **onAudioFocusChange** listener:

{% highlight java %}
this.onAudioFocusChange = (Function1)null.INSTANCE;
{% endhighlight %}

We can see that lambdas are translated to generic FunctionN classes where N is the number of parameters. Its implementation is hidden here and we'll need another tool to see exactly what is assigned but that's another story.

Let's check how **OnAudioFocusChangeListener** is really implemented:

{% highlight java %}
final class MainActivityKt$sam$OnAudioFocusChangeListener$4186f324 implements OnAudioFocusChangeListener {
   // $FF: synthetic field
   private final Function1 function;

   MainActivityKt$sam$OnAudioFocusChangeListener$4186f324(Function1 var1) {
      this.function = var1;
   }

   // $FF: synthetic method
   public final void onAudioFocusChange(int focusChange) {
      Intrinsics.checkExpressionValueIsNotNull(this.function.invoke(Integer.valueOf(focusChange)), "invoke(...)");
   }
}
{% endhighlight %}

And now let's see how it is used. The **requestAudioFocus** function:

{% highlight java %}
private final boolean requestAudioFocus() {
    Log.d(Companion.getTAG(), "requestAudioFocus() called");
    (...)
    if(var10001 != null) {
        Object var2 = var10001;
        var10001 = new MainActivityKt$sam$OnAudioFocusChangeListener$4186f324((Function1)var2);
    }

    int focusRequest = var10000.requestAudioFocus((OnAudioFocusChangeListener)var10001, 3, 1);
    Log.d(Companion.getTAG(), "granted = " + (focusRequest == 1));
    return focusRequest == 1;
}
{% endhighlight %}

The **abandonAudioFocus** function:

{% highlight java %}
private final boolean abandonAudioFocus() {
    Log.d(Companion.getTAG(), "abandonAudioFocus() called");
    (...)
    if(var10001 != null) {
        Object var2 = var10001;
        var10001 = new MainActivityKt$sam$OnAudioFocusChangeListener$4186f324((Function1)var2);
    }

    int focusRequest = var10000.abandonAudioFocus((OnAudioFocusChangeListener)var10001);
    Log.d(Companion.getTAG(), "granted = " + (focusRequest == 1));
    return focusRequest == 1;
}
{% endhighlight %}

You probably noticed the problematic line in both functions:

{% highlight java %}
var10001 = new MainActivityKt$sam$OnAudioFocusChangeListener$4186f324((Function1)var2);
{% endhighlight %}

What really happens is that our lambda / Function1 class is initialized in onCreate() but every time we pass it as a [SAM Interface](https://stackoverflow.com/questions/17913409/what-is-a-sam-type-in-java) to a function it gets wrapped in a new instance of a class implementing our listener interface **meaning two instances of the listener are created** and the **AudioManager API cannot remove on abandonAudioFocus() the listener previously created and passed to the first call of requestAudioFocus()** !!! As the original listener is never removed, it makes sense we get notified with [**AUDIO_FOCUS_LOSS**](https://developer.android.com/reference/android/media/AudioManager.html#AUDIOFOCUS_LOSS) on the first instance of our listener.

## The right way

Listeners need to stay anonymous inner classes, so here's the right way to declare it:

{% highlight kotlin %}
private lateinit var onAudioFocusChange: AudioManager.OnAudioFocusChangeListener
{% endhighlight %}

{% highlight kotlin %}
onAudioFocusChange = object : AudioManager.OnAudioFocusChangeListener {
    override fun onAudioFocusChange(focusChange: Int) {
        Log.d(TAG, "In onAudioFocusChange (${this.toString().substringAfterLast("@")}), focus changed to = $focusChange")
        when (focusChange) {
            AudioManager.AUDIOFOCUS_GAIN -> TODO("resume playing")
            AudioManager.AUDIOFOCUS_LOSS -> TODO("abandon focus and stop playing")
            AudioManager.AUDIOFOCUS_LOSS_TRANSIENT -> TODO("pause but keep focus")
         }
    }
}
{% endhighlight %}

Now the same instance class implementing [OnAudioChangeListener](https://developer.android.com/reference/android/media/AudioManager.OnAudioFocusChangeListener.html) is referenced by our **onAudioFocusChange** variable and is passed correctly to both functions, **requestAudioFocus** and **abandonAudioFocus** on **AudioManager**. Yay!

## The code example

You can check the generated bytecode and see the problem yourself [in this Github repository](https://github.com/galex/lambdas-listeners).

## Conclusion

With Great Power Comes Great Responsibility! Don't use lambdas **instead** of anonymous inner classes for **listeners**. I learned an important lesson here and I hope you learned from it too.

Cheers!
