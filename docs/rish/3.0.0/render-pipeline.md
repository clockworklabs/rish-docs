---
layout: docs
title: Render Pipeline
sections:
  - Elements Definitions
  - Mounting and Unmounting
  - Update Chain
order: 2
---

Rish handles how and when to update and render the Virtual Tree and Visual Trees, it also automatically pools all the elements to avoid garbage and optimize memory.
Each element in the tree will only be rendered when it's mounted (added to the tree) and after getting dirty.

## Elements Definitions
The `Render` method in `RishElement` returns an `Element`. This `Element` type defines a UI element in Rish, it holds the element type, its properties type, its properties values and everything that Rish needs to add that element to the tree, set it up and render it.

The `Children` type holds an ordered list of `Element`s.

## Mounting and Unmounting
When a RishElement is rendered for the first time, Rish calls the `Render` method, gets an instance of the returned element definition type from the pool and mounts it as a child of the RishElement that is being rendered.

If the new element is a RishElement, Rish sets up its properties, flags it as dirty. This makes Rish render the newly added element and the cycle repeats.

If it's instead a VisualElement, Rish sets up its UI Toolkit properties (name, class name and inline style), calls the `Setup` method with the Rish properties and attaches all the children. Each of these children are also Dirty when they get added to the tree and the cycle repeats.

When an element is no longer needed, all of its children are unmounted first, then the element also gets unmounted from the tree and they are all returned to the pool, ready to be reused.

### Callbacks
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

It's important to know that `IManualState.Restart` will be called right before the element gets reused, so we shouldn't use this method to unsubsribe from events or cancel actions since they'll keep happening after the element is unmounted.

## Update Chain
Elements will trigger an update chain any time they are dirty. An element is automatically flagged as dirty anytime its `Props` or `State` change. If an element is dirty, Rish will re render it by calling its `Render` function. The call to `Render` can result in elements to be reused with no changes, elements to be reused with changes (different properties), new elements to be added and/or elements to be removed. If any element has to be added or reused with different `Props`, then it will be flagged as dirty and Rish will call its `Render` function when it's its turn. That's why we call it an update chain. If at any level, we render an element and none of its children are changed, the update chain finishes. This way we can trust that Rish will only update and re-rendered the elements that strictly need to.

Let's work with an example:

{% highlight csharp %}
private partial class DirtyExample : RishElement<NoProps, DirtyExampleState>
{
    public DirtyExampleState {
        RegisterCallback<HoverStartEvent>(OnHoverStart);
        RegisterCallback<HoverEndEvent>(OnHoverEnd);
    }

    protected override Element Render() => Row.Create(
        children: new Children {
            H1.Create(text: value: "The element is:"),
            State.hovered ? P.Create(text: "hovered.") : P.Create(text: "not hovered..."),
            State.hovered ? Element.Null : P.Create(text: "yet.")
        });

    private void OnHoverStart(HoverStartEvent evt) => SetHover(true);
    private void OnHoverEnd(HoverEndEvent evt) => SetHover(false);
}
[RishValueType]
public struct DirtyExampleState {
    public bool hovered;
}
{% endhighlight %}

The element `DirtyExample` will result in `Row -> H1, P, P` being added to the tree with the resulting text "The element is: not hovered... yet". Once the element is hovered, its `State` changes, this flags the element as dirty, Rish re-renders it (calling the `Render` function again) and this time it results on `Row -> H1, P` with the text "The element is: hovered.". For this, Rish will reuse the `Row` element, resuse the `H1` (with no changes), reuse one of the `P` elements and unmount the other `P` element. Once the element is not hovered anymore, Rish will again reuse the `Row`, the `H1` (with no changes) and the `P` elements and mount a new `P` element.

### Element's Key
When rendering, Rish will always try to re-use an already mounted element if possible. If it can't find a current child of the necessary type, it will mount a new one from the pool. Obviously, it will never reuse, for example, an `H1` element if we need a `P` element. When there's more than one of the needed type already mounted as children (for example the `P` elements in the example above), Rish doesn't know which one to reuse and in which positions if we don't provide extra information. When creating Element definitions, we can assign a `ulong` key to each element and Rish will use this information to pick and reuse an element.

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
}
[RishValueType]
public struct DirtyExampleState {
    public bool hovered;
}
{% endhighlight %}

In this updated example, Rish knows which `P` to reuse and which `P` to mount and unmount.

### Manual Dirty
The user can manually flag an element as dirty from within by calling the `Dirty()` method. This might be useful for the few scenarios where an element needs to have instance state.

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

The `Dirty()` method has a bool argument (which is `false` by default) in case we need to force the element to be updated this frame and can't wait until next frame. In most cases we probably want to keep the default behavior to avoid re-rendering the same elements more than once in a frame.