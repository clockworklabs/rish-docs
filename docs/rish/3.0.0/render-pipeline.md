---
layout: docs
title: Render Pipeline
sections:
  - Mounting & Unmounting
  - Dirty Flag and Update Chain
order: 2
---

Rish is very smart on how and when to update and render the Virtual Tree. Rish also automatically pools all the elements to avoid garbage and optimize memory and processing usage.
Each element in the tree will be rendered when it's mounted (added to the tree) and after getting dirty.

## Mounting & Unmounting
When a RishElement returns another element definition in its `Render` method, Rish will get an instance of the corresponding element type from the pool and it will mount it as a child 
of the RishElement being rendered. If the new element being mounted is a VisualElement and in it's definition, the parent element defined a list of children, then all these child elements 
are also pulled from the pool and mounted as children of the new element.

When an element is no longer needed in the tree, it get's unmounted but all of its children are unmounted first.

In a `RishElement`, you can add callbacks for when the element is mounted and right before the element is unmounted by implementing the interface `IMountingListener`.

{% highlight csharp %}
private partial class FooElement : RishElement, IMountingListener
{
    void IMountingListener.ComponentDidMount() {
        Debug.Log("Element Mounted");
    }
    void IMountingListener.ComponentWillUnmount() { 
        Debug.Log("Element Unmounted");
    }

    protected override Element Render() => Element.Null;
}
{% endhighlight %}

In case your element needs some instance state (not Rish state, but actual C# instance variables or properties) that you need to restart so the next time the element gets reused (pulled from the pool), you can add `IManualState` listener.

{% highlight csharp %}
private partial class FooElement : RishElement, IManualState
{
    private HashSet<int> Indices { get; } = new();
    
    void IManualState.Restart() {
       Indices.Clear();
    }
}
{% endhighlight %}

## Dirty Flag & Update Chain
Elements will trigger an update chain any time they are dirty. An element is automatically flagged as dirty anytime its `Props` or `State` change. If an element is dirty, Rish will re render it by calling its `Render` function. The call to `Render` can result in elements to be reused with no changes, elements to be reused with changes (different `Props`), elements to be added and/or elements to be removed. If any element has to be added or reused with different `Props`, then it will be flagged as dirty and Rish will call its `Render` function. That's why we call it an update chain. If at any level, we render an element and all its children are reused with no changes, the update chain is stopped. This way we can rely Rish will only update the elements that need to be updated.

Let's work with an example:

{% highlight csharp %}
private partial class DirtyExample : RishElement<NoProps, DirtyExampleState>
{
    public DirtyExampleState {
        RegisterCallback<HoverStartEvent>(OnHoverStart);
        RegisterCallback<HoverEndEvent>(OnHoverEnd);
    }

    protected override Element Render() => Div.Create(children: new Children {
        H1.Create(text: value: "The element is:"),
        State.hovered ? P.Create(text: "hovered.") : P.Create(text: "not hovered..."),
        State.hovered ? Element.Null : P.Create(text: "yet.")
    });

    private void OnHoverStart(HoverStartEvent evt) => SetHover(true);
    private void OnHoverEnd(HoverEndEvent evt) => SetHover(false);
    private void SetHover(bool v) {
        var state = State;
        state.hovered = v;
        State = state;
    }
}
[RishValueType]
public struct DirtyExampleState {
    public bool hovered;
}
{% endhighlight %}

The element `DirtyExample` will show `Div -> H1, P, P` when mounted with the resulting text being "The element is: not hovered... yet". But when the element is hovered, its `State` will change, flagged as dirty and Rish updating it and calling its `Render` function again, resulting on `Div -> H1, P` with the text "The element is: hovered.". For this, Rish will reuse the `Div`, the `H1` (with no changes) and one of the `P` elements and unmount the other `P` element. When the user stops hovering the element, Rish will again reuse the `Div`, the `H1` (with no changes) and the `P` elements and mount a new `P` element.

### Element's Key
Whenever Rish needs to reuse elements, the first parameter it will consider is the elements types. Obviously, it will never reuse, for example, an `H1` element if we need a `P` element. But if we have two `P` elements, Rish doesn't know which one to reuse and in which positions if we don't provide extra information. When creating Element definitions, we can assign a `ulong` key to each element and Rish will use this information whenever it has to reuse an element.

{% highlight csharp %}
private partial class DirtyExample : RishElement<NoProps, DirtyExampleState>
{
    public DirtyExampleState {
        RegisterCallback<HoverStartEvent>(OnHoverStart);
        RegisterCallback<HoverEndEvent>(OnHoverEnd);
    }

    protected override Element Render() => Div.Create(children: new Children {
        H1.Create(text: value: "The element is:"),
        State.hovered ? P.Create(key: 1, text: "hovered.") : P.Create(key: 1, text: "not hovered..."),
        State.hovered ? Element.Null : P.Create(key: 2, text: "yet.")
    });

    private void OnHoverStart(HoverStartEvent evt) => SetHover(true);
    private void OnHoverEnd(HoverEndEvent evt) => SetHover(false);
    private void SetHover(bool v) {
        var state = State;
        state.hovered = v;
        State = state;
    }
}
[RishValueType]
public struct DirtyExampleState {
    public bool hovered;
}
{% endhighlight %}

In this updated example, Rish knows which `P` to reuse and which `P` to mount and unmount.

### Manual Dirty
The user can manually flag an element as dirty from within by calling the `Dirty()` method. This might be useful for the few scenarios where an element has instance state.

{% highlight csharp %}
private partial class ManualDirtyExample : RishElement, IManualState
{
    private HashSet<int> Indices { get; } = new();
    
    void IManualState.Restart() {
       Indices.Clear();
    }
    
    protected override Element Render() => H5.Create(text: $"Hovered by {Indices.Count} pointers.");

    private void OnHoverStart(HoverStartEvent evt) => AddPointer(evt.pointerId);
    private void OnHoverEnd(HoverEndEvent evt) => RemovePointer(evt.pointerId);
    private void AddPointer(int id) {
        Indices.Add(id);
        Dirty();
    }
    private void RemovePointer(int id) {
        Indices.Remove(id);
        Dirty();
    }
}
{% endhighlight %}