---
title: Responsive Style Sheets
slug: responsive-style-sheets
sections:
icon: pen-fancy
---

Responsive Style Sheets allow you to load certain USS style sheets depending on the app's screen width. They work similar to a min-width media query in CSS.

You can create a `ResponsiveStyleSheet` under `Roots -> ResponsiveStyleSheet` in the `Create` menu.

A `ResponsiveStyleSheet` is simply a list of `(int minWidth, StyleSheet styleSheet)` pairs. If the Min Width is <= 0, then the Style Sheet will always be loaded. If the Min Width is > 0, then it will only be loaded if the screen's width is equal or higher.

This means that you should use higher min width USS Style Sheets to override values set in previous ones.

<figure>
    <figcaption>styles-xs.uss (min-width: 0)</figcaption>
{% highlight css %}
.example {
    margin: 8px;
}
{% endhighlight %}
</figure>

<figure>
    <figcaption>styles-lg.uss (min-width: 1024)</figcaption>
{% highlight css %}
.example {
    margin: 30px;
}
{% endhighlight %}
</figure>

To be able to load and use `ResponsiveStyleSheets` you need to add a `RootsSetup` component to the same GameObject that has your `RishRoot` and `UIDocument`. Roots will handle adding and removing stylesheets if the app resizes.

`ResponsiveStyleSheets` are also convenient to pack style sheets together, even if you don't need any of the responsive logic. You can leave all the Min Sizes set to 0.

<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Be careful!</h4>
        <p>Roots adds all the style sheets on top of those added by Rish. This means that some USS values might get overriden.</p>
    </div>
</div>