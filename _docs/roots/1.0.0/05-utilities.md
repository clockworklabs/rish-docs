---
title: Utilities
slug: utilities
sections:
    - VisualElement Extensions
    - Style
    - Vector Swizzling
icon: toolbox
---

Roots comes with a couple of useful utilities.

## VisualElement Extensions
### Resolved Language Direction
`VisualElements` have a `languageDirection` property of type `public enum LanguageDirection{ Inherit, LTR, RTL }`. But when the value is `Inherit`, we don't know what the language direction is. Roots adds the `GetResolvedLanguageDirection` extension method which resolves the inherited directionlity.

## Style
Roots provides chain extension methods to set up inline `style`.

{% highlight csharp %}
Div.Create(
    style: StyleUtilites
        .FlexRow().
        .AlignItemsCenter()
        .Padding(16, 8)
    children: H3.Create(text: "Hello World"));
{% endhighlight %}

As always, the rule of favoring USS class names and style sheets over inline styling still apply, but these chain methods sometimes come handy.

## Vector Swizzling
We found ourselves needing to swizzle vector components all the time. So we created a bunch of extension methods just for that purpose and pack it as part of Roots.

{% highlight csharp %}
var vec0 = new Vector4(1, 2, 3, 4); // (1, 2, 3, 4)
var vec1 = vec0.sXY(); // (1, 2)
var vec2 = vec0.sX0Y(); // (1, 0, 2)
var vec3 = vec0.sZWWX(); // (3, 4, 4, 1)
{% endhighlight %}