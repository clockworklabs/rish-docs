---
layout: docs
title: Best Practices
sections:
  - Key
  - Props and State
  - Styling
  - Strings
  - Sappy
  - Compute Once
  - Conventions
order: 9
icon: award
---

## Key
When re-rendering an element, Rish will try to reuse all the children it can. If there's more than one child of a type, you should always provide a key to help Rish be extra efficient.

{% highlight csharp %}
public partial class InventoryElement : RishElement<InventoryProps>
{
    protected override Element Render() {
        var children = new Children();
        for(int i = 0, n = Props.pocketsCount; i < n ; i++) {
            children.Add(Pocket.Create(key: (ulong)i, inventoryId: Props.inventoryId, index: i));
        }

        return Grid.Create(children: children);
    }
}

[RishValueType]
public struct InventoryProps
{
    public ulong inventoryId;
    public int pocketsCount;
}
{% endhighlight %}

When a UI element represents or is tied to a game entity that has a unique ID, you should use that ID as the key.

{% highlight csharp %}
public partial class InventoryElement : RishElement<InventoryProps, InventoryState>
{
    void IPropsListener.PropsDidChange() {
        SetupPockets();
    }
    void IPropsListener.PropsWillChange() { }

    protected override Element Render() {
        var children = new Children();
        foreach(var pocketId in State.pocketsIds) {
            children.Add(Pocket.Create(key: pocketId, id: pocketId));
        }

        return Grid.Create(children: children);
    }

    private void SetupPockets() {
        var pocketsCount = InventoryState.Get(Props.inventoryId)?.PocketsCount ?? 0;
        if(pocketsCount <= 0) {
            SetPocketsIds(RishList<ulong>.Null);
            return;
        }

        using(ManagedContext.New()) {
            var pocketsIds = new RishList<ulong>();
            for(var i = 0; i < pocketsCount; i++) {
                var id = PocketState.Get(Props.inventoryId, i)?.Id ?? 0;
                if(id <= 0) continue;
                pocketsIds.Add(id);
            }

            SetPocketsIds(pocketsIds);
        }
    }
}

[RishValueType]
public struct InventoryProps
{
    public ulong inventoryId;
}

[RishValueType]
public struct InventoryState
{
    public RishList<ulong> pocketsIds;
}
{% endhighlight %}

## Props and State
Members of Props and State (and all `RishValueType`s) should follow these guidelines:
- fields for raw data that is passed down;
- properties for inferred data;
- methods for more expensive (>O(1)) inferred data;
- avoid reference types as much as possible (to help guarantee UI determinism).

### Callbacks in Props
Callbacks should never affect the visual look of an Element. For that reason, they're ignored by default when comparing Props. If the only property that changed is a callback, we don't want to start a whole Update chain to get the exact same visual result. To make things worse, how C# handles lambdas internally is a little unpredictable. For these reasons, it's always better to pass down new lambdas to member methods that call callbacks in Props rather than passing down the callback from Props directly.

Consider the following example:

{% highlight csharp %}
public partial class Button : RishElement<ButtonProps>
{
    protected override Element Render() => AbstractButton.Create(
        action: Props.action,
        content: Div.Create(
            className: "button",
            style: new Style {
                backgroundColor = Props.color
            },
            children: H5.Create(text: Props.label)));
}

[RishValueType]
public struct ButtonProps
{
    public Color color;
    public RishString label;
    public Action action;
}
{% endhighlight %}

We have a Button element wrapping an abstract button that handles player input. We pass `Props.action` callback directly to the `AbstractButton` child.

{% highlight csharp %}
public partial class PingPong : RishElement<NoProps, PingPongState>
{
    protected override Element Render() => Col.Create(
        children: new Children {
            H3.Create(text: $"The counter: {State.counter}"),
            Button.Create(
                action: State.pong ? DecreaseCounter : IncreaseCounter,
                color: Color.blue,
                label: "Change counter" 
            )
        });
    
    private void IncreaseCounter() => SetCounter(State.counter + 1);
    private void DecreaseCounter() => SetCounter(State.counter - 1);

    private void SetCounter(int value) {
        RishSetCounter(value);
        if(value >= 10) {
            SetPong(true);
        } else if(value <= 0) {
            SetPong(false);
        }
    }
}

[RishValueType]
public struct PingPongState
{
    public int counter;
    public bool pong;
}
{% endhighlight %}

We have an app that should ping pong a counter between 0 and 10. The `Button` visual properties (color and label) stay always the same but the `action` changes.

If you play with this app, the counter will start at 0 and every time you press the button will increase by 1. 0, 1, 2... 8, 9, 10... 11, 12, 13. It didn't ping pong back to 0! Why? Rish is trying to be efficient and when `PingPong` gets re-rendered, `Button` never gets dirty, because the callback is being ignored when comparing Props and because we passed `Props.action` directly to `AbstractButton`, `AbstractButton` is always calling that same callback we passed down first.

The solution is extremely simple (and Rishenerator makes it even easier).

{% highlight csharp %}
public partial class Button : RishElement<ButtonProps>
{
    protected override Element Render() => AbstractButton.Create(
        action: Action, // Action method is auto-generated by Rishenerator
        content: Div.Create(
            className: "button",
            style: new Style {
                backgroundColor = Props.color
            },
            children: H5.Create(text: Props.label)));
}

[RishValueType]
public struct ButtonProps
{
    public Color color;
    public RishString label;
    public Action action;
}
{% endhighlight %}

This works because when Rish compares the `ButtonProps`, even though the comparer returns true, it updates the Props value, it just doesn't flag the element as dirty, so `Button` has the updated callback in `Props.action` and the `Action` method calls it.

## Styling
When it comes to VisualElements and best styling practices, Rish doesn't have much to say. UI Toolkit is king here and you should follow their recommended best practices, like favoring USS class names over inline styling, for example.

## Strings
Strings are reference types, but since they're immutable it should be safe to use them in Props and State, right? Well, yeah, but you still shouldn't. Enter `RishString`.

`RishString` is just a wrapper of `string`, there's nothing really special about it but it has a couple of nice things:
- It's a value type, which means you can use create a `RishList<RishString>` (`RishList` generic type parameter has a value type constraint).
- It provides an `IsEmpty` property that uses `string.IsNullOrWhiteSpace`, so instead of calling `string.IsNullOrWhiteSpace(str)` everywhere, we can simply to do `str.IsEmpty`.
- `null`, empty and white space strings all evaluate to true when compared to each other.
- Length returns 0 for `null` or whitespace strings and we can avoid a lot of `null` checks (and `NullReferencePointerExceptions`).
- Indexers and Remove function return `string.Empty` instead of `null` so we can avoid a lot of `null` checks (and `NullReferencePointerExceptions`).

## Sappy

## Compute Once

## Conventions