---
title: Tooltips
slug: tooltips
sections:
    - Context
icon: comment-dots
---

Because UI Toolkit lacks a `z-index` property and the visual order is determined by the order in the hierarchy, we have to get creative when we want elements to be rendered on top of everything. Tooltips are one such situation.

## Context
A `TooltipContext` ancestor is necessary for `Tooltips` to work. When a tooltip is needed, it will add it at the end of its hierarchy (on top of everything else) and will handle its positioning.

It will only show one tooltip at a time.

{% highlight csharp %}
TooltipsContext.Create(
    name: "tooltips",
    children: new Children {
        // Some Children
            // ...
                // ...
                    Tooltip.Create(
                        // ...
                    )
        // More Children
    });
{% endhighlight %}

### Props
- `bool forceFit`: If true, tooltips are forced to fit within the `TooltipContext` content rect.
- `bool hideTooltips`: If true, no tooltip will show.
- `Action<bool> onShow`: Callback that gets called when a tootlips is shown or hidden.
- `VisualAttributes visualAttributes`: Styling information. Expanded in `Create` method.
- `Children children`: Children.

## `Tooltip`
The `Tooltip` element is a foundational RishElement. In the `Render` method, it transparently passes the `content` from Props (`protected override Element Render() => Props.content;`) but it listens to pointer hover events and communicates with a `TooltipContext` ancestor to show the `tooltip` in the right place.

{% highlight csharp %}
TooltipsContext.Create(
    name: "tooltips",
    children: new Children {
        // Some Children
            // ...
                // ...
                    Tooltip.Create(
                        // ...
                    )
        // More Children
    });
{% endhighlight %}

<div class="alert alert-success" role="alert">
    Rootstrap provides a <code>SimpleTooltip</code> wrapper.
</div>

### Props
- `Element content`: The child content.
- `Element tooltip`: The tooltip element to show when hovering the content.
- `bool ignoreFit`: This can override if the context has `forceFit` set to `true`.
- `float enterDelay`: Delay to wait before showing the tooltip.
- `float exitDelay`: Delay to wait before hiding the tooltip.