---
title: VisualElements
slug: visualelements
sections:
  - Visual Tree
  - Styling
  - Props
  - Composition (Children)
  - Input Handling
  - Lifecycle and Callbacks
  - The Render Process
icon: eye
---

## Visual Tree
Rish uses **UI Toolkit** as its rendering backend. Every piece of UI that actually appears on screen is a standard Unity `VisualElement` added to the Visual Tree.

Nothing in the Visual Tree built by Rish is "magic", custom or special. The difference lies in how we build and update that tree.

For a `VisualElement` to be usable in Rish, it must implement the `IVisualElement` interface.

Because Rish needs to manage the lifecycle and updates of these elements, every implementation requires two specific internal components: a **Bridge** and a **PickingManager**.

#### The Standard Boilerplate
Here is the minimal setup required to create a valid Rish VisualElement:

{% highlight csharp %}
private partial class Div : VisualElement, IVisualElement
{    
    // 1. The Bridge: Connects Rish logic to the Unity VisualElement
    private Bridge Bridge { get; }
    Bridge IVisualElement.Bridge => Bridge;

    // 2. The PickingManager: Handles input detection (Raycasting)
    private PickingManager PickingManager { get; }
    PickingManager ICustomPicking.Manager => PickingManager;

    // 3. Constructor: Initialize components
    public Div()
    {
        Bridge = new Bridge(this);
        PickingManager = new RectPickingManager(Bridge);
    }
    
    // 4. Setup: Called when Props change
    void IVisualElement.Setup() { }
}
{% endhighlight %}

## Styling
Just like HTML elements have an `id`, UI Toolkit elements have a `name`. This is useful for styling but also for debugging and querying elements in the UI Toolkit Debugger.

{% highlight csharp %}
protected override Element Render() => Div.Create(name: "main-container");
{% endhighlight %}

UI Toolkit uses **USS** (a subset of CSS). You can style elements by assigning class names or by using inline styles directly in C#.

{% highlight csharp %}
protected override Element Render() => Div.Create(
    className: "card", // Reference to USS class
    style: new Style { // Inline style
        backgroundColor = Color.red,
        marginTop = 10
    });
{% endhighlight %}

<div class="callout-block callout-block-warning">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Best Practice</h4>
        <p>Favor <strong>USS style sheets</strong> (via <code>className</code>) over inline styles. Unity optimizes style sheets better (and supports live reloading of USS assets) and it keeps your style and layout logic separate from your code.</p>
        <p>Use <strong>Inline Styles</strong> only when the style depends on dynamic data from Props or State.</p>
    </div>
</div>

#### Dynamic Styling Example
{% highlight csharp %}
private partial class InlineStyleExample : RishElement<InlineStyleExampleProps>
{    
    protected override Element Render() => Div.Create(
        className: "position-absolute",
        style: new Style { 
            top: Props.margin.top,
            right: Props.margin.right,
            bottom: Props.margin.bottom,
            left: Props.margin.left
        });
}

[RishValueType]
public struct InlineStyleExampleProps {
    public Margin margin;
}
{% endhighlight %}

## Props
Just like `RishElements`, `VisualElements` can receive strongly-typed **Props**.

When defining a VisualElement with props, you implement `IVisualElement<T>`. The `Setup` method is where you use these props and apply them to the underlying UI Toolkit element.

{% highlight csharp %}
private partial class ExampleProps : VisualElement, IVisualElement<ExampleProps>
{
    private Bridge<ExampleProps> Bridge { get; }
    Bridge IVisualElement<ExampleProps>.Bridge => Bridge;

    // ...

    void IVisualElement<ExampleProps>.Setup(ExampleProps props) {
        style.backgroundColor = props.color;
    }
}

[RishValueType]
public struct ExampleProps {
    public Color color;
}
{% endhighlight %}

<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title">Note</h4>
        <p>The purpose of <code>Setup</code> is strictly to configure the element's own properties (color, text, texture, etc.). You should <strong>not</strong> add or remove children inside <code>Setup</code>.</p>
    </div>
</div>

## Composition (Children)
Rish handles the hierarchy for you. You pass children into a VisualElement via the `children` parameter in the `Create` method.

{% highlight csharp %}
protected override Element Render() => Div.Create(
    children: new Children {
        Div.Create(name: "Header"),
        Div.Create(name: "Body", children: new Children {
             Div.Create(name: "Content A"),
             Div.Create(name: "Content B")
        }),
        Div.Create(name: "Footer")
    });
{% endhighlight %}

## Input Handling
To handle user input (clicks, hovers, drags) correctly, Rish needs to know if a pointer is "inside" the element. This is handled by the `PickingManager`.
- `RectPickingManager`: Returns true for any point inside the element's layout rectangle. (Standard behavior).
- `DiscardPickingManager`: Always returns false. Use this for elements that should be "invisible" to the mouse (pass-through).

<div class="alert alert-info" role="alert">
    Future versions of Rish will include PickingManagers that support rounded corners and transparency checks.
</div>

## Lifecycle and Callbacks
Rish provides interfaces to hook into the lifecycle of a `VisualElement`.

### Mounting Events (IMountingListener)
Useful for initialization logic or event subscription.

{% highlight csharp %}
void IMountingListener.ComponentDidMount() {
    Debug.Log("Element added to the Visual Tree");
}
void IMountingListener.ComponentWillUnmount() { 
    Debug.Log("Element about to be removed");
}
{% endhighlight %}

### Pooling Reset (IManualState)
If your VisualElement uses instance state that needs to be reset before the element is reused, you can use `IManualState`.

{% highlight csharp %}
private partial class SpriteSheet : VisualElement, IVisualElement<SpriteSheetProps>, IManualState
{
    private List<Sprite> Frames { get; set; }
    
    // Called right BEFORE the element is reused from the pool
    void IManualState.Restart() {
        Frames.Clear();
    }

    // ...
}
{% endhighlight %}

### Style Listeners
You can react to styling changes by implementing specific listeners:
- `INameListener`: Called when `name` changes.
- `IClassNameListener`: Called when the class list changes.
- `IStyleListener`: Called when inline styles change.

{% highlight csharp %}
private partial class FooElement : VisualElement, IVisualElement, INameListener, IClassNameListener, IStyleListener
{
    void INameListener.DidChange(Name value) {
        Debug.Log($"The element is now called {value}.");
    }
    void IClassNameListener.DidChange(ClassName value) {
        Debug.Log($"The element has {value.Count} class names.");
    }
    void IStyleListener.DidChange(Style value) {
        Debug.Log($"The element inline style changed.");
    }

    // ...
}
{% endhighlight %}

## The Render Process
When Rish renders a `VisualElement`:
- **Sync:** It updates `name`, `className`, and `style` (in that order).
- **Setup:** It calls `Setup(props)` (only if Props have changed).
- **Children:** It reconciles and updates all children.

<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title">Important</h4>
        <p>Layout and style updates happen asynchronously on UI Toolkit's side. If you query <code>resolvedStyle</code> (e.g., <code>element.resolvedStyle.width</code>) immediately after a render, it may not yet reflect the new values.</p>
    </div>
</div>