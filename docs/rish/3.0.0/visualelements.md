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
  - Render Function
order: 3
---

## Visual Tree
Rish uses UI Toolkit as its rendering layer. This means that every visual element that gets rendered to the screen is a UIToolkit's VisualElement added to a UIToolkit's Visual Tree. You can debug this tree using UI Toolkit's Debugger. Nothing in the Visual Tree built by Rish is any different than any other UI Toolkit's tree, the difference is on how we build and update such tree.

For a VisualElement to be able to be used in Rish, it has to implement the `IVisualElement` interface. Like any regular VisualElement, it can have a name (equivalent to id in HTML and CSS), it can be styled (through USS files and Rish will also provide a convenient way to inline style elements, similar to HTML) and it can have children elements.

{% highlight csharp %}
private partial class Div : VisualElement, IVisualElement
{    
    void IVisualElement.Render() { }
}
{% endhighlight %}

## Name
In HTML, all elements can have an id attribute. The analogous in UI Toolkit, is the `name` property. Whenever we create an element definition for a Visual Element, we can specify its name. The name should be unique and it can be helpful to identify the different elements when debugging and for styling.

{% highlight csharp %}
private partial class Foo : RishElement
{    
    protected override Element Render() => Div.Create(name: "foo");
}
{% endhighlight %}

## Styling
UI Toolkit uses a subset of CSS called [USS](https://docs.unity3d.com/Manual/UIToolkits.html). You can style elements using USS files and class names or by inline styling when creating the different element definitions.

{% highlight csharp %}
private partial class Foo : RishElement
{    
    protected override Element Render() => Div.Create(
        className: "foo",
        style: new Style { 
            backgroundColor: Color.red
        },
        children: Div.Create(className: new ClassName {
            "class-0",
            "class-1",
            "class-2"
        }));
}
{% endhighlight %}

As a general rule, just like in HTML, we would recommend favoring styling using style sheet files over inline styling. A classic example of a necessary use of inline styling, is to style an element based on Props:

{% highlight csharp %}
private partial class InlineStyleExample : RishElement<InlineStyleExampleProps>
{    
    protected override Element Render() => Div.Create(
        className: "absolute-position",
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
All VisualElements can have children (in future revisions we might consider a way to specify if a Visual Element should be able to have or not children). When defining a VisualElement, we can pass one or more children.

{% highlight csharp %}
private partial class ChildrenExample : RishElement
{    
    protected override Element Render() => Div.Create(
        children: new Children {
            Div.Create(name: "Child 0"),
            Div.Create(name: "Child 1"),
            Div.Create(name: "Child 2", children: new Children {
                Div.Create(name: "Child 0 of Child 2"),
                Div.Create(name: "Child 1 of Child 2"),
            }),
            Div.Create(name: "Child 3", children: Div.Create("Child 0 of Child 3")),
            Div.Create(name: "Child 4"),
        });
}
{% endhighlight %}

## DOMDescriptor
It's very common for a RishElement to act as a wrapper for a VisualElement and expect all the styling information necessary to style the VisualElement. For these scenarios, we can use `DOMDescriptor`, which containes `name`, `className` and `style`. If we use the `DOMDescriptor` attribute, these properties will be automatically expanded in the `Create` function.

{% highlight csharp %}
private partial class Example : RishElement
{    
    protected override Element Render() => Example.Create(name: "container", className: "class-name", style: new Style { }, children: Div.Create());
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
`VisualElements` can have Props just like `RishElements`. The Props type of `VisualElements` doesn't need the `RishValueType` attribute and the `Render` function will receive the `Props` as a parameter:
{% highlight csharp %}
private partial class ExampleProps : VisualElement, IVisualElement<ExampleProps>
{    
    void IVisualElement<ExampleProps>.Render(ExampleProps props) {
        Debug.Log($"The color is: {props.color}");
    }
}

[RishValueType]
public struct ExampleProps {
    public Color color;
}
{% endhighlight %}

## Render Function
The purpose of the `Render` function in a `VisualElement` is to setup the element. We don't expect to add or create any children, we should just setup everything we need to make this element look like it should based on what we received through Props.