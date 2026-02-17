---
layout: docs
title: Quick Start
sections:
  - Why Rish?
  - Elements Composition
  - Inputs and Data Flow
  - It's Just C#
  - Roots
order: 0
icon: handshake
---

Welcome to **Rish**! Rish is a declarative UI library for Unity that uses **UI Toolkit** as its render layer.

If you are familiar with React or other declarative UI frameworks, you will feel right at home. Rish (loosely) follows the React paradigm: you tell the computer what to do, not how to do it.

## Why Rish?
**Declarative:** You define the desired state of the UI, and Rish handles the updates.

**Best of Both Worlds:** It combines the benefits of Immediate Mode (code-driven, logical) with Retained Mode (stateful, efficient).

**Deterministic:** Implemented properly, your UI becomes a pure function of your game state. Always in sync and (hopefully 🤞) bug-free.

## Elements Composition
Rish apps are built using Elements (a piece of the UI that has it's own logic and appearence). (Note: While React calls them "Components," we stick to "Elements" to align with UI Toolkit's naming conventions).

There are two distinct types of elements in Rish:

### VisualElements (The Render Layer)
These are standard UI Toolkit elements that implement the `IVisualElement` interface to be used in Rish. They are the actual objects added to Unity's UI Toolkit Visual Tree.

Best Practice: Avoid creating complex logic here. Use these primarily for rendering.

### RishElements (The Logic Layer)
These are the core building blocks of your application (similar to React Components). They inherit from `RishElement` and they must implement a `Render()` method that returns an `Element`.

Here are two basic example of a RishElement:

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

Elements are designed to be composed. You can nest elements inside one another.

{% highlight csharp %}
public partial class App : RishElement {
    protected override Element Render() => Div.Create(
        children: new Children {
            WelcomeTitle.Create(),
            WelcomeMessage.Create()
        });
}
{% endhighlight %}

<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>The Virtual Tree</h4>
        <p>All RishElements exist in a "Virtual Tree". This is a superset of the Visual Tree. When Rish renders, it "squashes" the RishElements down, leaving only the VisualElements in the actual UI Toolkit hierarchy. You can inspect the final result using the UI Toolkit Debugger.</p>
    </div>
</div>

## Inputs and Data Flow
Rish is strictly typed. To pass data into your elements or manage internal data, you use **Props** and **State**. Both must be `structs` flagged with the `[RishValueType]` attribute.

### Props (External Data)
Props allow you to pass data _down_ from a parent element to a child.

{% highlight csharp %}
public partial class Card : RishElement<CardProps> {
    protected override Element Render() => Div.Create(
        className: "card",
        children: new Children
        {
            H1.Create(text: Props.title),
            P.Create(text: Props.message)
        });
}
[RishValueType]
public struct CardProps {
    public RishString title;
    public RishString message;
}

// Usage in a parent element
public partial class App : RishElement {
    protected override Element Render() => Col.Create(
        children: new Children
        {
            Card.Create(title: "Card 1", message: "This is a card"),
            Card.Create(title: "Card 2", message: "This is another different card")
        });
}
{% endhighlight %}

### State (Internal Data)
State is data managed _internally_ by the element itself. When state changes, Rish automatically detects it and re-renders the element.

{% highlight csharp %}
public partial class Counter : RishElement<NoProps, CardState> {
    protected override Element Render() => Div.Create(
        children: new Children
        {
            H1.Create(text: $"The count is {State.counter}"),
            Button.Create(action: AddOne, children: "+1")
        });

    private void AddOne() => SetCounter(State.counter + 1);
}
[RishValueType]
public struct CounterState {
    public int counter;
}
{% endhighlight %}

### Reactivity and Determinism
Rish automatically detects changes in **Props** or **State** and flags the element as "dirty," triggering a re-render.

To achieve a Deterministic UI, treat your `Render` method as a Pure Function:

$$ UI = f(Props,State) $$

If you ensure your `Render` method relies _only_ on Props and State, your UI will always be predictable and in sync with your data.

## It's Just C#
Rish is 100% C#. There is no custom templating language or XML to learn. No weird setups to bridge different technologies stacks. You have direct access to your game data and Unity's APIs, you can use `if` statements, `for` loops, LINQ... you name it.

{% highlight csharp %}
public partial class ItemCard : RishElement<ItemCardProps> {
    protected override Element Render() {
        // Access global game data
        var item = StaticData.GetItem(Props.id);

        // Standard C# control flow
        if(item.hidden) {
            return Element.Null;
        }

        // Complex logic using LINQ
        var recipesCount = StaticData.CraftingRecipes.Count(r => r.output == Props.id);

        return Card.Create(title: item.Name, message: $"There are {recipesCount} ways of crafting {item.Name}.");
    }
}
[RishValueType]
public struct ItemCardProps {
    public int id;
}
{% endhighlight %}

## Roots
Rish is a UI library, not a rigid or opinionated framework. We're trying not to impose anything on you and that's why it comes with virtually no elements. You have the freedom to implement your library of elements the way that best suits your needs.

However, if you want a head start, we have created **Roots**, a collection of ready-to-use elements built with Rish.

[Check out the Roots documentantion](/docs/roots/1.0.0/quick-start)