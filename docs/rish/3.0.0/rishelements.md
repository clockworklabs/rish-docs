---
layout: docs
title: RishElements
sections:
  - Virtual Tree
  - Inputs
  - DOMDescriptor
  - Wrap VisualElements
  - Callbacks
  - UIToolkit Events
order: 4
icon: brain
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

Props and State types must be struct value types and have the `RishValueType` attribute. The convention is to name them "[ElementName]Props" and "[ElementName]State" respectively. For example:

{% highlight csharp %}
public partial class Foo : RishElement<FooProps, FooState>
{    
    protected override Element Render() => Element.Null;
}

[RishValueType]
public struct FooProps { }

[RishValueType]
public struct FooState { }
{% endhighlight %}

### Default Value
To define default values for Props or State, you need to implement a static property of the same type with the `Default` attribute. Default Props values will be used for all properties that are not specified when defining an element and State will be initialized with the default values when mounting the element.
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
    private static FooProps Default => new() {
        flag1 = true,
        n = 1
    };
}

[RishValueType]
public struct FooState {
    public int counter; // Will start at 1

    [Default]
    private static FooState Default => new() { counter = 1 };
}

public partial class Example : RishElement { 
    protected override Element Render() => Foo.Create(n: 3); // Will create a Foo with flag0 = false, flag1 = true and n = 3.
}
{% endhighlight %}

## Wrap VisualElements
It's very common for a RishElement to act as a wrapper for a VisualElement and expect the children and styling information necessary to style the VisualElement through Props. For these scenarios, you can use `DOMDescriptor`, which containes `name`, `className` and `style`. The `DOMDescriptor` attribute expands the styling attributes in the `Create` function.

{% highlight csharp %}
private partial class Example : RishElement
{    
    protected override Element Render() => Container.Create(
        name: "container",
        className: "class-name",
        style: new Style {
            // ...
        },
        children: Div.Create());
}

private partial class Container : RishElement<ContainerProps>
{    
    protected override Element Render() => Div.Create(
        name: Props.descriptor.name,
        className: Props.descriptor.className + "another-class-name",
        style: Props.descriptor.style,
        children: Props.children);
}

[RishValueType]
public struct Container {
    [DOMDescriptor]
    public DOMDescriptor descriptor;
    public Children children;
}
{% endhighlight %}

## Callbacks
Rish offers many interfaces that RishElements can implement to listen for important events.

### Mounting and Unmounting
A `RishElement` can implement the `IMountingListener` interface to receive callbacks for when the it gets mounted and right before it gets unmounted.

{% highlight csharp %}
private partial class FooElement : RishElement, IMountingListener
{
    void IMountingListener.ComponentDidMount() {
        Debug.Log("Element mounted");
    }
    void IMountingListener.ComponentWillUnmount() { 
        Debug.Log("Element will be unmounted");
    }

    protected override Element Render() => Element.Null;
}
{% endhighlight %}

For cases when your element needs some instance state (not Rish state, but actual C# instance variables or properties) that you need to restart before the element gets reused (from the pool), you can add `IManualState` listener.

{% highlight csharp %}
private partial class FooElement : RishElement, IManualState
{
    private HashSet<int> Indices { get; } = new();
    
    void IManualState.Restart() {
       Indices.Clear();
    }

    // ...
}
{% endhighlight %}

It's important to know that `IManualState.Restart` will be called right before the element gets reused, so we shouldn't use this method to unsubsribe from events or cancel actions since they'll keep happening after the element is unmounted (until is mounted again).

#### Advanced Unmounting
Any time an element is not needed anymore, the unmounting process begins immediately. Usually this means the element will be removed from the tree instantly but sometimes we need to delay it (to play an outro animation, for example). For these cases, Rish offers the `ICustomUnmountListener` interface. This interface provides two methods: `UnmountRequested` for when the unmounting process begins and `Unmounted` for when the element is actually finally removed from the tree.

If a RishElement implements this interface, Rish will call the `UnmountRequested` method and won't remove the element from the tree until `CanUnmount()` is manually called within the RishElement.

{% highlight csharp %}
public partial class DelaySampleElement : RishElement, IMountingListener, ICustomUnmountListener
{
    void IMountingListener.ComponentDidMount() {
        Debug.Log("1. Element was added to the tree.");
    }
    void IMountingListener.ComponentWillUnmount() {
        Debug.Log("4. Element is about to be removed from the tree.");
    }

    void ICustomUnmountListener.UnmountRequested() {
        Debug.Log("2. Element is not needed anymore. We'll wait for 5 seconds before unmounting.");

        Countdown.Start(5, CountdownIsOver);
    }
    void ICustomUnmountListener.Unmounted() {
        Debug.Log("5. Element was removed from the tree.");
    }

    protected override Element Render() => Element.Null;

    private void CountdownIsOver() {
        Debug.Log("3. 5 seconds countdown is over. We can unmount now.");
        CanUnmount();
    }
}
{% endhighlight %}

### Props
A `RishElement` can implement the `IPropsListener` interface to receive callbacks when the Props are changing.

{% highlight csharp %}
public partial class FooElement : RishElement<FooProps>, IPropsListener
{
    void IPropsListener.PropsDidChange() {
        Debug.Log($"Element props changed. The id is: {Props.id}.");
    }
    void IPropsListener.PropsWillChange() { 
        Debug.Log("Element props will change.");
    }

    protected override Element Render() => Element.Null;
}

[RishValueType]
public struct FooProps {
    public int id;
}
{% endhighlight %}

`PropsDidChange` is called after the element is mounted and Props are set for the first time and every time the element gets Dirty with different Props. `PropsWillChange` is called before the Props value changes and right before unmounting the element.

We also have `IPropsListener<P>` for when we need to compare the previous Props value. It's useful to avoid doing duplicated Setup work when not needed.

{% highlight csharp %}
public partial class FooElement : RishElement<FooProps, FooState>, IPropsListener<FooProps>
{
    void IPropsListener<FooProps>.PropsDidChange(FooProps? prev) {
        if(prev.HasValue && prev.Value.id == Props.id) return;

        Setup();
    }
    void IPropsListener<FooProps>.PropsWillChange() { }

    protected override Element Render() => P.Create(
        name: Props.descriptor.name,
        className: Props.descriptor.className,
        style: Props.descriptor.style,
        text: $"The item is called {State.name}");

    private void Setup() {
        SetName(ItemDesc.Get(Props.id).Name);
    }
}

[RishValueType]
public struct FooProps {
    [DOMDescriptor]
    public DOMDescriptor descriptor; // descriptor can change and it won't trigger another setup

    public int id; // if id changes, Setup will be called
}

[RishValueType]
public struct FooState {
    public RishString name;
}
{% endhighlight %}

And, lastly, we have `IAllPropsListener`, to receive callbacks every time a Props value is set, even if the element is not getting dirty by it (the Props comparison returns true).

## UIToolkit Events
At some point, one of your RishElements will likely have to listen to some event coming from UI Toolkit (to respond to pointer events, for example). To keep things simple, Rish provides an extremely similar API to the one in VisualElements.

RishElements can subscribe to events with `RegisterCallback` and unsubscribe from them with `UnregisterCallback`. Rish will add the callbacks to the first VisualElement descendant.

{% highlight csharp %}
public partial class FooElement : RishElement<NoProps, FooState>
{
    public FooElement() {
        RegisterCallback<PointerEnterEvent>(OnPointerEnter);
        RegisterCallback<PointerLeaveEvent>(OnPointerLeave);
    }

    protected override Element Render() => P.Create(text: $"This element is {(State.hovered ? "being hovered" : "not hovered")}.");

    private void OnPointerEnter(PointerEnterEvent evt) => SetHovered(true);
    private void OnPointerLeave(PointerLeaveEvent evt) => SetHovered(false);
}

[RishValueType]
public struct FooState {
    public bool hovered;
}
{% endhighlight %}

You can also create a `ToolkitManipulator` (similar to UI Toolkit's `Manipulator`) and add it to your RishElement with `AddManipulator` and remove it with `RemoveManipulator`.

{% highlight csharp %}
public partial class FooElement : RishElement
{
    public FooElement() {
        AddManipulator(new ClickManipulator());
    }

    // ...
}

public partial class ClickManipulator : ToolkitManipulator
{
    protected override void RegisterCallbacksOnTarget()
    {
        // Add callbacks to target
    }

    protected override void UnregisterCallbacksFromTarget()
    {
        // Remove callbacks from target
    }
}
{% endhighlight %}