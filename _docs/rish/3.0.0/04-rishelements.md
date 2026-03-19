---
title: RishElements
slug: rishelements
sections:
  - Inputs
  - VisualAttributes
  - Wrap VisualElements
  - Callbacks
  - UIToolkit Events
icon: brain
---

While `VisualElements` handle the actual rendering on screen, `RishElements` handle the logic, state, and structure of your application.

All UI elements are added to a **Virtual Tree** managed by Rish.
- **VisualElements:** Exist in both the Virtual Tree and the UI Toolkit Visual Tree.
- **RishElements:** Exist _only_ in the Virtual Tree.

To create a new RishElement, you inherit from `RishElement` and implement the `Render` method. A `RishElement` always has exactly one child (the element returned by `Render`). If an element shouldn't have a child, you can return `Element.Null`.

### Types
There are three main base classes for `RishElement`, depending on your data needs:

- `RishElement`: For elements with no Props or State.
- `RishElement<Props>`: For elements that receive data from parents.
- `RishElement<Props, State>`: For elements that receive data and manage internal state.

<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Node</h4>
        <p>Internally, every RishElement has a Props type. The base <code>RishElement</code> simply inherits from <code>RishElement&lt;NoProps&gt;</code>.</p>
    </div>
</div>

## Inputs: Props and State
Our goal is a **Deterministic UI**. We treat UI elements as **Pure Functions**:

$$ UI = f(Props,State) $$

- **Props:** Data passed down from a parent.
- **State:** Data managed internally by the element.

The `Render` function should rely _only_ on Props and State. Every change to either will automatically trigger a re-render.

### Defining Props and State
Props and State must be `struct` types with the `[RishValueType]` attribute. The convention is to name them `[ElementName]Props` and `[ElementName]State`.

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

### Default Values
Both Props and State can have default values.
- **Props Defaults:** Used if the parent does not provide a value.
- **State Defaults:** Used to initialize the state when the element is first mounted.

To define default values, there are two options:

#### Individual Default Values
Add the `[DefaultValue]` attribute to specific fields that need a default value.

{% highlight csharp %}
[RishValueType]
public struct FooProps {
    [DefaultValue(true)]
    public bool isVisible;
    public int count;
}
{% endhighlight %}

The `[DefaultValue]` attribute can only accept compile-time constant expressions. For instances where this is not enough, you can use the `[DefaultCode]` attribute and Rishenerator will copy and paste the string provided.

{% highlight csharp %}
[RishValueType]
public struct FooProps {
    public bool isVisible;
    [DefaultCode("UnityEngine.Random.Range(0, 10)")]
    public int count;
}
{% endhighlight %}

<div class="callout-block callout-block-success">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-check"></i>Best Practice</h4>
        <p>For Props specifically, it's always advisable to use these individual default values (above) over the <code>[Default]</code> property explained below. It results in both better programmer experience and better performance.</p>
    </div>
</div>

#### Default Property
Implement a static property of the same type with the `[Default]` attribute inside your struct.

{% highlight csharp %}
[RishValueType]
public struct FooProps {
    public bool isVisible;
    public int count;

    [Default]
    private static FooProps Default => new() {
        isVisible = true,
        count = 1
    };
}
{% endhighlight %}

## Wrapping VisualElements
A very common pattern is creating a `RishElement` that wraps a `VisualElement` (like a custom Card or Container) but allows the parent to style it.

Instead of passing every style property manually, you can use `VisualAttributes`. This struct contains `name`, `className`, and `style`.

{% highlight csharp %}
private partial class Example : RishElement
{    
    protected override Element Render() => AlertContainer.Create(
        style: new Style { // Style doesn't exist in Props but the [Expand] attribute generates this
            maxWidth = Length.Percent(70),
            margin = 16
        },
        children: Col.Create(
            children: new Children
            {
                H4.Create(text: "Alert title"),
                P.Create(text: "Alert body")
            }));
}

private partial class AlertContainer : RishElement<AlertContainerProps>
{    
    protected override Element Render() => Div.Create(
        name: Props.attributes.name,
        className: Props.attributes.className + "alert",
        style: Props.attributes.style,
        children: Props.children);
}

[RishValueType]
public struct AlertContainer {
    [Expand]
    public VisualAttributes attributes;
    public Children children;
}
{% endhighlight %}

## Lifecycle and Callbacks
Rish provides interfaces to hook into the lifecycle of a RishElement.

### Mounting Events (IMountingListener)
Implement `IMountingListener` to know when an element enters or leaves the tree.

{% highlight csharp %}
private partial class FooElement : RishElement, IMountingListener
{
    void IMountingListener.ElementDidMount() {
        Debug.Log("Element mounted");
    }
    void IMountingListener.ElementWillUnmount() { 
        Debug.Log("Element will be unmounted");
    }

    protected override Element Render() => Element.Null;
}
{% endhighlight %}

### Pooling Reset (IManualState)
If your VisualElement holds instance state that you must reset before the element is reused, you can use `IManualState`.

{% highlight csharp %}
private partial class FooElement : RishElement, IManualState
{
    private HashSet<int> Indices { get; } = new();
    
    // Called right BEFORE the element is reused from the pool
    void IManualState.Restart() {
       Indices.Clear();
    }

    // ...
}
{% endhighlight %}

<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-triangle-exclamation"></i>Be Careful!</h4>
        <p><code>IManualState.Restart</code> will be called right before the element gets reused from the Pool. Do not use <code>Restart</code> to unsubscribe from events or cancel actions. Use <code>ElementWillUnmount</code> for that.</p>
    </div>
</div>

### Delayed Unmounting
Sometimes you need to delay removal, for example, to play an Outro Animation.

If you implement `ICustomUnmountListener`:
1. Rish calls `UnmountRequested()`.
2. Rish waits and keeps the element in the tree.
3. You must manually call `CanUnmount()` when the element is ready to be unmounted.

{% highlight csharp %}
public partial class DelaySampleElement : RishElement, IMountingListener, ICustomUnmountListener
{
    void IMountingListener.ElementDidMount() {
        Debug.Log("1. Element was added to the tree.");
    }
    void IMountingListener.ElementWillUnmount() {
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

### Props Listeners
If you need to execute logic when Props change (e.g., fetching data based on an ID), implement `IPropsListener`.

Rish provides three flavors:

#### 1. `IPropsListener` (Basic)
Notifies you that props have changed and will change.

{% highlight csharp %}
public partial class FooElement : RishElement<FooProps>, IPropsListener
{
    // Called after mounting and every time props change
    void IPropsListener.PropsDidChange() {
        Debug.Log($"Element props changed. The new id is: {Props.id}.");
    }
    // Called right before changing props
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

#### 2. `IPropsListener<T>` (Comparison)
Allows you to compare previous and current props to avoid unnecessary work.

{% highlight csharp %}
public partial class FooElement : RishElement<FooProps, FooState>, IPropsListener<FooProps>
{
    void IPropsListener<FooProps>.PropsDidChange(FooProps? prev) {
        if(prev.HasValue && prev.Value.id == Props.id) return;

        Setup();
    }
    void IPropsListener<FooProps>.PropsWillChange() { }

    protected override Element Render() => P.Create(
        name: Props.attributes.name,
        className: Props.attributes.className,
        style: Props.attributes.style,
        text: $"The item is called {State.name}");

    private void Setup() {
        SetName(ItemDesc.Get(Props.id).Name);
    }
}

[RishValueType]
public struct FooProps {
    [Expand]
    public VisualAttributes attributes; // visualAttributes can change and it won't trigger another setup

    public int id; // if id changes, Setup will be called
}

[RishValueType]
public struct FooState {
    public RishString name;
}
{% endhighlight %}

#### 3. `IAllPropsListener<T>` (Comparison)
Notifies you every time props are set, even if the values are identical (and the element is not marked Dirty).

## UIToolkit Events
RishElements can listen to UI Toolkit events (like clicks or hovers) just like VisualElements.

<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Note</h4>
        <p>Since RishElements don't exist in the Visual Tree, Rish attaches these callbacks to the first VisualElement descendant.</p>
    </div>
</div>

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

### Manipulators
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

These are useful to throw new types of events or to standarize responses to certain events.