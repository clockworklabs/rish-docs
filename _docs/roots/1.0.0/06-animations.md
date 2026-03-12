---
title: Animations
slug: animations
sections:
    - MotionDiv
    - FadeDiv
icon: wand-magic-sparkles
---


## `MotionDiv`
A `MotionDiv` serves as a container, like a regular `Div`, but it's a RishElement that allows you to animate most of its USS properties.

You can pass down styling information through class names and inline style, but these are going to be used only to initialize the `MotionDiv` because once mounted, all of its USS properties are internally managed. The way to change properties is through the `animate` property, which defines the target values and how to transition to them.

It also has an `initial` property to define initial values and an `exit` property to play outro animations when the element is unmounted.

{% highlight csharp %}
MotionDiv.Create(
    className: "progress-bar",
    initial: new Initial
    {
        width = 0
    },
    animate: new Target
    {
        width = Length.Percent(t * 100),
        transition = new Transition
        {
            width = Spring.Bouncy
        }
    },
    exit: new Target {
        opacity = 0
    },
    children: P.Create(text: "Loading..."));
{% endhighlight %}

You can also pass `false` in `initial` to skip the intro animation for a property and jump directly to the value defined in `animate`.

{% highlight csharp %}
MotionDiv.Create(
    initial: new Initial
    {
        width = false
    },
    animate: new Target
    {
        width = Length.Percent(t * 100) // Won't have an intro animation when mounted.
    });
{% endhighlight %}

In `Target` you can define transitions per property as well as a general one.

{% highlight csharp %}
MotionDiv.Create(
    className: "animated",
    initial: new Initial
    {
        width = 0,
        height = 0
    },
    animate: new Target
    {
        width = Length.Percent(100),
        height = Length.Percent(100),
        transition = new Transition
        {
            width = Spring.Bouncy,
            height = new Spring {
                stiffness = 400,
                damping = 35,
                mass = 3.5f,
            }
        }
    },
    exit: new Target {
        width = 0,
        opacity = 0,
        transition = new Transition {
            all = Spring.Fast,
            opacity = new Tween(0.25f, Ease.Linear)
        }
    },
    children: P.Create(text: "Loading..."));
{% endhighlight %}

<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-triangle-exclamation"></i>Exit Animations</h4>
        <p>At the moment, if an element has many siblings, the position of the element being unmounted might be wrong during the outro animation.</p>
        <p>We'll fix this in a future version.</p>
    </div>
</div>

### Props
- `Initial initial`: Initial values for intro animations played when the element is mounted.
- `Target animate`: Target values.
- `Target exit`: Exit values for outro animations played when the element is unmounted.
- `Action onAnimateComplete`: Callback that gets called when the `animate` animation is completed.
- `Action onExitComplete`: Callback that gets called when the `exit` animation is completed.
- `VisualAttributes visualAttributes`: Styling information. Expanded in `Create` method.
- `Children children`: Children.

## `FadeDiv`
A `FadeDiv` wraps around a `MotionDiv` and it simplifies fade in and fade out logic and animations.

### Props
- `bool visible`: Whether the element is visible or not.
- `bool fadeOnUnmount`: If true, the fade out animation is played before unmounting.
- `bool keepChildrenWhenInvisible`: If true, all children persist mounted while the element is invisible.
- `FadeDiv.Preset preset`: An animation preset (`Slow`, `Fast` or `Immediate`) to be used if duration is not provided.
- `float? duration`: Fade animation duration.
- `float? minOpacity`: Target opacity when `visible` is `false`. Assumed 0 if no value is provided.
- `float? maxOpacity`: Target opacity when `visible` is `true`. Assumed 1 if no value is provided.
- `Action onAnimateComplete`: Callback that gets called when the `animate` animation is completed.
- `VisualAttributes visualAttributes`: Styling information. Expanded in `Create` method.
- `Children children`: Children.