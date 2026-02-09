---
layout: docs
title: VisualElements
sections:
  - Visual Tree
  - Name
  - Styling
  - Children
  - DOMDescriptor
  - Props
  - Render
  - Bridge
  - Pointer Detection
  - Styled Props
order: 3
---

## Visual Tree
Rish uses UI Toolkit as its rendering layer. This means that every visual UI piece that gets rendered to the screen is a UIToolkit's VisualElement being added to a UIToolkit's Tree. You can debug this tree using UI Toolkit's Debugger. Nothing in the Visual Tree built by Rish is any different or especial than any other UI Toolkit tree, the difference is on how we build and update such tree.

For a VisualElement to be able to be used in Rish, it has to implement the `IVisualElement` interface.

{% highlight csharp %}
private partial class Div : VisualElement, IVisualElement
{    
    void IVisualElement.Setup() { }
}
{% endhighlight %}

## Name
In HTML, all elements can have an id attribute. The analogous in UI Toolkit is the `name` property. Whenever we create an element definition for a Visual Element, we can specify its name. The name should be unique and it can be helpful both to identify and style elements.

{% highlight csharp %}
private partial class Foo : RishElement
{    
    protected override Element Render() => Div.Create(name: "foo");
}
{% endhighlight %}

## Styling
UI Toolkit uses a subset of CSS called [USS](https://docs.unity3d.com/Manual/UIToolkits.html). You can style elements using USS files and class names or via inline styles when creating the different element definitions.

{% highlight csharp %}
private partial class Foo : RishElement
{    
    protected override Element Render() => Div.Create(
        className: "foo",
        style: new Style { 
            backgroundColor: Color.red
        },
        children: Div.Create(
            className: new ClassName {
                "class-0",
                "class-1",
                "class-2"
            }));
}
{% endhighlight %}

As a general rule, just like in HTML, we would recommend favoring the use of style sheets over inline styling (note that Unity recommends using USS files when possible because it performs better also). A classic example of a necessary use of inline styling, is to style an element based on Props (or State):

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

## Children
All VisualElements can have children (in future revisions we might consider a way to specify if a Visual Element should be able to have 0, 1, ..., n or any number of children). When defining a VisualElement, we can define children elements and pass them down using the children property.

{% highlight csharp %}
private partial class ChildrenExample : RishElement
{    
    protected override Element Render() => Div.Create(
        children: new Children {
            Div.Create(name: "Child 0"),
            Div.Create(name: "Child 1"),
            Div.Create(
                name: "Child 2",
                children: new Children {
                    Div.Create(name: "Child 0 of Child 2"),
                    Div.Create(name: "Child 1 of Child 2"),
                }),
            Div.Create(name: "Child 3", children: Div.Create("Child 0 of Child 3")),
            Div.Create(name: "Child 4"),
        });
}
{% endhighlight %}

## DOMDescriptor (Move to RishElement)
It's very common for a RishElement to act as a wrapper for a VisualElement and expect all the styling information necessary to style the VisualElement. For these scenarios, we can use `DOMDescriptor`, which containes `name`, `className` and `style`. If we use the `DOMDescriptor` attribute, these properties will be automatically expanded in the `Create` function.

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
        className: Props.descriptor.className,
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

## Props
Similarly to `RishElements`, `VisualElements` can have Props. They also need to be a struct type and have the `RishValueType` attribute. The `Setup` method will receive them as a parameter:
{% highlight csharp %}
private partial class ExampleProps : VisualElement, IVisualElement<ExampleProps>
{    
    void IVisualElement<ExampleProps>.Setup(ExampleProps props) {
        Debug.Log($"The color is: {props.color}");
    }
}

[RishValueType]
public struct ExampleProps {
    public Color color;
}
{% endhighlight %}

The purpose of the `Setup` method in a `VisualElement` is to use the Props to setup the element. We don't expect to add or create any children, we should just setup everything we need to make this element look like it should based on what we received through Props.

## Render
When rendering a VisualElement, Rish first sets up the `name`, `className` and `style` attributes (in that order), then it calls the `Setup` method (only if the Props changed) and, lastly, it adds all the children (and the update chain continues).

It's important to understand that the visual update and layout of the VisualElements still happen on UI Toolkit's side and won't happen immediately. For this reason, for example, the `resolvedStyle` of a VisualElement won't reflect the new values until UI Toolkit processes it.

## Bridge
When implementing the `IVisualElement` or `IVisualElement<PropsType>` interface, you'll need to provide a `Bridge`. This is the bridge between Rish and UI Toolkit and it handles setting up the VisualElement and updating it only when its inputs are dirty.

The `Bridge` class has a constructor that requires the VisualElement as an argument. This is the usual setup:
{% highlight csharp %}
private partial class Div : VisualElement, IVisualElement
{
    private Bridge Bridge { get; }
    Bridge IVisualElement.Bridge => Bridge;
    
    public Div()
    {
        Bridge = new Bridge(this);
    }
    
    // ...
}

private partial class ComplexVisualElement : VisualElement, IVisualElement<ComplexVisualElementProps>
{
    private Bridge<ComplexVisualElementProps> Bridge { get; }
    Bridge IVisualElement<ComplexVisualElementProps>.Bridge => Bridge;
    
    public Div()
    {
        Bridge = new Bridge<ComplexVisualElementProps>(this);
    }
    
    // ...
}
{% endhighlight %}

## Pointer Detection
In any real world application we'll need UI elements to be able to detect input from the user. To handle pointers input correctly, we use Picking Managers. The `IVisualElement` interface, implements the `ICustomPicking` interface which forces us to provide a `PickingManager`. A Picking Manager implements a `Raycast` method and determines if a point is within the element or not to be used in input events. For now, we have `DiscardPickingManager` (which always returns false, for elements that should never react to input) and `RectPickingManager`.

`RectPickingManager` returns true for all points within the layout rect of a VisualElement. In future versions, we'll add support to more advance Picking Manager to allow ignoring points over rounded corners or transparent parts of an image. 

The usual setup looks like this:
{% highlight csharp %}
private partial class Div : VisualElement, IVisualElement
{
    private Bridge Bridge { get; }
    Bridge IVisualElement.Bridge => Bridge;

    private PickingManager PickingManager { get; }
    PickingManager ICustomPicking.Manager => PickingManager;

    public Div()
    {
        Bridge = new Bridge(this);
        PickingManager = new RectPickingManager(Bridge);
    }
    
    // ...
}
{% endhighlight %}

## Styled Props
Since VisualElements are styled units of UI and we use Props to set them up, it's natural that we should be able to set some Props via USS style sheets too. For this, we have the `IStyledProps` interface.

This API will change and improve in a future version and much of the boilerplate code will be autogenerated by Rishenerator. For this reason, we won't cover it in detail for now.