---
title: Labels
slug: labels
sections:
  - Inputs
icon: align-left
---

`Label` is the _only_ element shipped with Rish. It's a foundational VisualElement. It inherits from `UnityEngine.UIElements.Label`.

A `string` or `RishString` can be implicitly converted into an `Element`. So, similarly to HTML, you can treat text as a UI Element.

<figure>
    <figcaption>HTML</figcaption>
{% highlight html %}
<div class="container">Hello, world.</div>
{% endhighlight %}
</figure>

<figure>
    <figcaption>C# Rish</figcaption>
{% highlight csharp %}
Div.Create(className: "container", children: "Hello, world.");
{% endhighlight %}
</figure>

We recommend creating wrappers for different label styles (like `Body`, `H1`, `H2`...).

## Inputs
- `RishString text`: The text to render.
- `LengthRange? widthRange`: Optional width size range. If set, UI Toolkit will use for the reported preferred size in the layouting phase.
- `LengthRange? heightRange`: Optional height size range. If set, UI Toolkit will use for the reported preferred size in the layouting phase.
- `bool? enableRichText`: Whether or not rich text tags get parsed or not.
- `bool? parseEscapeSequences`: Wheter or not escape sequences get parsed or not.
- `Action<bool> onElided`: Callback that reports when the text is ellided or not.