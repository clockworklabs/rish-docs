---
layout: docs
title: RishElements
sections:
  - Virtual Tree
  - Inputs
  - Props
  - State
  - DOMDescriptor
  - Children
  - Interface Callbacks
order: 4
---

## Virtual Tree
All UI elements are added to a Virtual Tree managed by Rish. VisualElements are also added to UI Toolkit's Visual Tree. RishElements are only added to the Virtual Tree. To create a new RishElement type, you need to inherit from one of the `RishElement`s classes and implement the `Render` function. A RishElement will always have the element it returns in the `Render` function as its only child. If an element shouldn't have a child, you can return `Element.Null`.

At the moment, there's no way to visualize the Virtual Tree structure but we'll add editor tools in future releases.

### Types
For practical purposes, there are three `RishElement` types:
- `RishElement<P>`, for elements with Props of type P;
- `RishElement<P, S>`, for elements with Props of type P and State of type S;
- `RishElement`, for elements with no Props or State.

In actuality, every RishElement has a Props type and `RishElement` inherits from `RishElement<NoProps>`.

## Inputs
Our goal is to create a deterministic UI and we think of UI elements as pure functions. The inputs of a RishElement function are its Props and State.

Props come from "above": another RishElement created a definition with specific properties that are passed down. State, on the other hand, is entirely internal.

The Render function should only use Props and State to help guarantee that the UI remains deterministic. Every change to Props or State will always trigger the element to be re-rendered.

For some advanced elements, you may need to use some other input (other than Props and State). In those cases, you'll have to be extra cautious on how you implement the Render function and manually flag your element as dirty.

## Props
Props types must be struct value types and have the `RishValueType` attribute. The convention is to name them "[ElementName]Props". For example:

{% highlight csharp %}
public partial class Foo : RishElement<FooProps>
{    
    protected override Element Render() => Element.Null;
}

[RishValueType]
public struct FooProps { }
{% endhighlight %}

### Members
Props can have all the members as you want. We recommend sticking to fields for data that is passed down, properties for inferred data from the fields and methods for more expensive (>O(1)) inferred data. For example:

{% highlight csharp %}
[RishValueType]
public struct FooProps {
    public bool flag0;
    public bool flag1;
    public uint n;

    public bool flag => flag0 || flag1;

    public ulong GetFactorial() {
        ulong result = 1;
        for (var i = 1; i <= n; i++)
        {
            result *= i;
        }
        return result;
    }
}
{% endhighlight %}

### Comparison
Rish needs to compare Props to determine if a RishElement is dirty and should be re-rendered or not. When using Rishenerator (which we highly suggest), this is automatically handled by Rish. This auto-generated comparer will compare all fields and you can add attributes to specify how to compare each field:
- `IgnoreComparison`: To ignore fields when comparing.
- `EqualityOperatorComparison`: To use `==` when comparing.
- `EqualsMethodComparison`: To compare using the `Equals` method.
- `EpsilonComparison`: Only for `float` fields, to compare using Unity's `Mathf.Approximately` function.
If no comparison attribute is provided, the default behavior is as follows:
- For reference type fields:
    - Delegate types: To ignore them.
    - Other reference types: To compare references.
- For value types: To use a manual comparer, if defined (more on this on the next section), or to perform a low-level binary comparison in every other case.

#### Manual Comparers
Some extra special value types might need extra care when compared. In this cases, we can implement a static predicate `(T, T) -> Bool` function flagged with the `Comparer` attribute. If Rish finds one of these defined, it will use it to compare value type fields.

For example, Rish's `Element` is a value type that simply holds an id for an internally managed reference type. Two element definitions could be equal to each other even if their ids are different, so we defined the following Comparer to use the Equals function of the internal reference type instead:
{% highlight csharp %}
[Comparer]
private static bool Equals(Element a, Element b)
{
    var aSet = a.Valid;
    var bSet = b.Valid;
    if (aSet ^ bSet)
    {
        return false;
    }
    if (!aSet)
    {
        return true;
    }
    
    var aDefinition = a.GetDefinition();
    var bDefinition = b.GetDefinition();

    var aDisposed = aDefinition == null;
    var bDisposed = bDefinition == null;
    if (aDisposed || bDisposed)
    {
        return false;
    }

    return aDefinition.Equals(bDefinition);
}
{% endhighlight %}

### Default Value
To define default values for Props, you need to implement a static property of the same type as the Props with the `Default` attribute. For example:
{% highlight csharp %}
public partial class Foo : RishElement<FooProps> { 
    // ...
}

[RishValueType]
public struct FooProps {
    public bool flag0;
    public bool flag1;
    public uint n;

    [Default]
    private static FooProps Default => new() { flag1 = true };
}

public partial class Example : RishElement { 
    protected override Element Render() => Foo.Create(); // Will create a Foo with flag1 true.
}
{% endhighlight %}