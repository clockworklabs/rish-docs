---
layout: docs
title: Visual Manipulation
sections:
  - Implementation
  - The Lifecycle
  - Performance
order: 7
icon: sliders
---

Rish provides an API for `RishElements` to modify the `style` and properties of their descendant `VisualElements` directly.

While standard data flow involves passing Props down, Visual Manipulation is sometimes very useful (and sometimes essential). For example, in Roots, we use it in `Rows` and `Cols` to add a gap between elements by manipulating the margin of the necessary descendants.

## Implementation
To turn a `RishElement` into a Visual Manipulator, implement the `IVisualManipulator` interface. This requires two methods:
1. **Evaluate:** Determines which descendants should be manipulated.
2. **Manipulate:** Applies the actual logic (style changes).

Below is a simplified implementation of a `Row` element that adds a gap between its children:

{% highlight csharp %}
public partial class Row : RishElement<RowProps>, IManualState, IVisualManipulator
{
    // We know our direct descendant will always be the same Div. So we cache it.
    private VisualElement _visualChild;
    private VisualElement VisualChild => _visualChild ??= GetVisualChild();

    void IManualState.Restart() {
        _visualChild = null;
    }

    // 1. SELECT: We want to manipulate the Row itself AND its immediate children
    bool IVisualManipulator.Evaluate(VisualElement descendant) => descendant == VisualChild || descendant.parent == VisualChild;

    // 2. ACT: Apply margins based on the 'gap' prop
    void IVisualManipulator.Manipulate(VisualManipulationPhase phase, IManipulable descendant)
    {
        // Optimization: If no gap is needed, do nothing
        if (Mathf.Approximately(Props.gap, 0)) return;

        var style = descendant.style;

        // Skip hidden elements
        if(style.display == DisplayStyle.None) return;

        var margin = Props.gap * 0.5f;

        // Apply negative margin to the container to offset outer padding
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
public struct RowProps {
    public float gap;
    public Children children;
}
{% endhighlight %}

## The Lifecycle
Visual Manipulation occurs in two distinct phases, passed as an argument to the `Manipulate` method:

1. `BubbleUp` Phase:
  - Trigger: A descendant element is mounted/re-rendered.
  - Flow: The descendant notifies its ancestors, asking, "Do any of you want to style me?".
2. `TrickleDown` Phase:
  - Trigger: The Manipulator (Parent) is re-rendered (e.g., Props.gap changes).
  - Flow: The parent reaches down to all previously evaluated descendants and re-applies the logic.

## Performance
While powerful, Visual Manipulation should be used sparingly. Don't turn every element into a manipulator. However, Rish optimizes this process heavily and it is still fairly performant.

The `Manipulate` method provides an `IManipulable` wrapper instead of the raw VisualElement. Rish uses this to **cache and batch** modifications.

If you have multiple nested Manipulators (e.g., a Row inside a Col inside a Row), Rish collects all style changes from all manipulators in the chain and applies them in a single pass at the end.

You can access and modify `name` and `style` directly (just like a standard `VisualElement`).

{% highlight csharp %}
descendant.name = "updated-name";
descendant.style.backgroundColor = Color.red;
{% endhighlight %}

Because `ClassName` is a Pointer Value Type, you cannot simply create a copy of the current value. Instead, you must use the provided helper methods within a valid Managed Context.

{% highlight csharp %}
using(ManagedContext.New())
{
    var className = descendant.CloneClassName();
    className.Add("my-new-class"); 
    descendant.SetClassName(className);
}
{% endhighlight %}