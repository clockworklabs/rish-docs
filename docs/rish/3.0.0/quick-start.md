---
layout: docs
title: Quick Start
sections:
  - Creating and nesting elements
  - Elements Inputs
  - It's just C#
  - Roots
order: 0
---

Welcome to the Rish documentation! Rish is a declarative UI library for Unity. It follows React's paradigm, so it's probably useful to know the basics and get familiar with the ideas behind React. Rish uses UI Toolkit as the render layer.

By declarative UI, we mean you tell the computer what to do, not how to do it. It has all the benefits of an immediate mode UI, but it's also stateful. Rish automatically detects which elements should be updated and re-rendered. That's why we say it combines the best of both retained and immediate modes.

Implemented properly, you should get a deterministic UI: Given a specific state, there's only one posible UI result. Something quite hard to achieve with different architectures.

## Creating and nesting elements
Rish apps are made out of elements (in React, they're called Components, but we've decided to stick to Elements following UI Toolkit's naming). An element is a piece of the UI that has it's own logic and appearence.

### Visual and Virtual Trees
In Rish there are two types of Elements: `RishElements` and `VisualElements`. `VisualElements` are UI Toolkit elements and as such, they're the pieces that get added and rendered to UI Toolkit's visual Tree. In HTML and React, these would be the DOM Element objects (like `div`, `button`, `a`). For a `VisualElement` to be able to be used in Rish, it has to implement the interface `IVisualElement`. All `VisualElements` can have a name, a list of USS class names, inline style and children passed down when defined. The recomendation would be to not create a massive collection of `VisualElements` since they are necessary only for the render layer, it's better to leave the heavy lifting and more complex logic for `RishElements` (unless not possible).

`RishElements` are the core pieces in Rish. In React, these would be the React Elements, but there are some key differences. `RishElements` are classes that inherit from `RishElement` and they need to implement a `Render` method that returns an `Element`.

{% highlight csharp %}
public partial class WelcomeTitle : RishElement {
    protected override Element Render() => H1.Create(text: "Hello, World");
}
{% endhighlight %}

{% highlight csharp %}
public partial class WelcomeMessage : RishElement {
    protected override Element Render() => P.Create(text: "You're gonna love Rish! I promise.");
}
{% endhighlight %}

Elements can be nested in another element.

{% highlight csharp %}
public partial class App : RishElement {
    protected override Element Render() => Div.Create(children: new Children {
        WelcomeTitle.Create(),
        WelcomeMessage.Create()
    });
}
{% endhighlight %}

All `RishElements` are added to a virtual tree. The visual tree is a subset of the virtual tree, where all `RishElements` have been squashed down. In the virtual tree, each `RishElement` is a parent of the element defined in the `Render` function and `VisualElements` are parents of the entities passed down as children. Currently there's no way to preview and inspect the virtual tree, but you can use inspect the visual tree using the `UI Toolkit Debugger`.

## Elements Inputs
### Props
When defining elements, you can pass down data to them, we call this data "properties". Both `RishElements` and `VisualElements` can receive properties. Everything in Rish is strictly typed and properties aren't the exception. For an element to have properties, it has to define what they are. Properties types must be structs and they have to be flagged with the attribute `RishValueType`.

{% highlight csharp %}
public partial class Card : RishElement<CardProps> {
    protected override Element Render() => Div.Create(className: "card", children: new Children
    {
        H1.Create(text: Props.title),
        P.Create(text: Props.message)
    });
}
[RishValueType]
public struct CardProps {
    public FixedString64Bytes title;
    public FixedString4096Bytes message;
}

public partial class App : RishElement {
    protected override Element Render() => Div.Create(children: new Children
    {
        Card.Create(title: "Card 1", message: "This is a card"),
        Card.Create(title: "Card 2", message: "This is another different card")
    });
}
{% endhighlight %}

Properties can only be passed down from defining elements.

### State
Alongside properties, `RishElements` can also have internal state. State must be a struct value type flagged with the `RishValueType` attribute. The main difference with properties, is that state is completely internal to the element.

{% highlight csharp %}
public partial class Counter : RishElement<NoProps, CardState> {
    protected override Element Render() => Div.Create(children: new Children
    {
        H1.Create(text: $"The count is {State.counter}"),
        Button.Create(action: AddOne, children: "+1")
    });

    private void AddOne() {
       var state = State;
       state.count += 1;
       State = state;
    }
}
[RishValueType]
public struct CounterState {
    public int counter;
}
{% endhighlight %}

### Dirty Elements
Rish will automatically detect changes in `Props` or `State` and flag the corresponding element as dirty, triggering it to be re-rendered. There are ways of manually flagging an element as dirty but they should only be used when extrictly necessary if you know what you're doing. We'll cover how to do this in a future section.

### Determinism
Since our goal is to accomplish a deterministic UI, we'll think about all are UI pieces (all the different elements) as functions and their properties and state as the inputs of said function. Thinking about our UI that way, our elements should be pure functions.

This means, in the `Render` method, we should only use `Props` and `State` and always return the exact same value given the exact same inputs. Implemented this way, most of UI programming asociated head aches will be gone.

## It's just C#
Rish is implemented entirely in C# and all your UI written using solely C# (besides USS style sheets). This means you can just do pretty much anything you can think of. There's no weird new syntax or language to learn, no faulty bridges between different technology stacks and no missing features due to architecture incompatibilities. You can access Unity's API, all your game data, you can use `if` statement, `for` loops, LINQ... it's all there for you to use. And it's fast.

{% highlight csharp %}
public partial class ItemCard : RishElement<ItemCardProps> {
    protected override Element Render() {
        var item = StaticData.GetItem(Props.id);
        if(item.hidden) {
            return Element.Null;
        }

        var recipesCount = StaticData.CraftingRecipes.Count(r => r.output == Props.id);

        return Card.Create(title: item.Name, message: $"There's {recipesCount} ways of crafting this item.");
    }
}
[RishValueType]
public struct ItemCardProps {
    public int id;
}
{% endhighlight %}

## Roots
Rish is just a UI library, it's not a framework. We're trying to not impose anything on you and that's why it comes with virtually no elements out of the box. You can build all the elements you want/need on top of it the way better suits your needs. But if you want a head start or a reference to look at, we have implemented a whole colection of elements called [Roots][roots-docs].

[roots-docs]: /docs/roots/1.0.0/quick-start