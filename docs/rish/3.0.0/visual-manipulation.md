---
layout: docs
title: Visual Manipulation
order: 7
icon: sliders
---

Rish provides an API for RishElements to visually manipulate descendant VisualElements. This is useful when a parent RishElement wants to change the styling of any arbitrary VisualElement descendant. In Roots, for example, we use it in Rows and Cols to add a gap between elements by manually adding a margin to the necessary descendants (this is necessary because UI Toolkit still doesn't have support for the gap property in USS).

To turn a RishElement into a Visual Manipulator, it has to implement the `IVisualManipulator` interface.

{% highlight csharp %}
public partial class Row : RishElement<RowProps>, IVisualManipulator
{
    private VisualElement _visualChild;
    private VisualElement VisualChild => _visualChild ??= GetVisualChild();

    bool IVisualManipulator.Evaluate(VisualElement descendant) => descendant == VisualChild || descendant.parent == VisualChild;

    void IVisualManipulator.Manipulate(VisualManipulationPhase phase, IManipulable descendant)
    {
        if (Mathf.Approximately(Props.gap, 0)) return;

        var style = descendant.style;

        if(style.display == DisplayStyle.None) return;

        var margin = Props.gap * 0.5f;
        if (descendant.element == VisualChild)
        {
            margin = -margin;
        }

        style.marginLeft = margin;
        style.marginRight = margin;
    }

    protected override Element Render() => Div.Create(className: "flex-row", children: Props.children);
}

[RishValueType]
public struct FooProps {
    public float gap;
    public Children children;
}
{% endhighlight %}

In the `Evaluate` method we need to return true for all elements we want to manipulate and in the `Manipulate` method we perform the changes. The first argument of the `Manipulate` method is phase: it can be either `BubbleUp` (if the descendant was rendered and is now reporting to its ancestors) or `TrickleDown` (if the RishElement was re-rendered and has to modify its descendants again).

`IManipulable` is a wrapper around a `VisualElement`. It holds an accessible reference to the `VisualElement` but, whenever possible, you should modify the `IManipulable` because it caches all the modifications and performs them all at once at the end of the manipulation chain (in cases where we have multiple nested Visual Manipulators).

In `IManipulable`, you can access and modify `name` and `style` directly, just like in `VisualElement`. To modify `className`, you have `CloneClassName` and `SetClassName` methods. This is necessary because `ClassName` is a Reference Value Type.