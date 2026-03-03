---
title: Best Practices
slug: best-practices
sections:
  - Performance & Optimization
  - Architecture and Logic
  - Style Guide And Conventions
icon: award
---

To get the most out of Rish—both in terms of performance and developer sanity—we recommend following these guidelines.

## Performance & Optimization
### Keys
When rendering a list of children, Rish tries to reuse existing elements. Every time you have more than one child of the same type, you should provide a key to help Rish be extra efficient.

{% highlight csharp %}
public partial class InventoryElement : RishElement<InventoryProps>
{
    protected override Element Render() {
        var children = new Children();
        for(int i = 0, n = Props.pocketsCount; i < n ; i++) {
            // Each Pocket is being identified by its position in the list of children
            children.Add(Pocket.Create(key: (ulong)i, inventoryId: Props.inventoryId, index: i));
        }

        return Grid.Create(children: children);
    }
}
{% endhighlight %}

In cases where the list of children order or size changes, Rish might update the wrong element with new data, which is inefficient. This is why we recommend to use a consistent unique ID (e.g., EntityID) for the key whenever is possible.

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

### Compute Once
The `Render` method is called every time an element is dirty. Avoid computations inside of it.

Instead, compute the desired values when their dependencies (probably Props) change, and store the result in State.

##### Bad: Computing in Render
{% highlight csharp %}
public partial class ItemsList : RishElement<ItemsListProps>
{    
    protected override Element Render()
    {
        // We're doing all of this every time this element gets rendered. Even if only title changed.
        var items = new Children();
        foreach (var id in Props.ids)
        {
            if (items.Count > 0)
            {
                items.Add(Separator.Create(key: (ulong)items.Count));
            }
            items.Add(Item.Create(key: (ulong)id, index: id));
        }

        return Col.Create(
            className: "items-list",
            children: new Children
            {
                H4.Create(text: Props.title),
                items
            });
    }
}
[RishValueType]
public struct ItemsListProps
{
    public RishString title;
    public RishList<int> ids;
}
{% endhighlight %}

##### Good: Computing Only When Needed

{% highlight csharp %}
public partial class ItemsList : RishElement<ItemsListProps, ItemsListState>, IPropsListener<ItemsListProps>
{
    void IPropsListener<ItemsListProps>.PropsDidChange(ItemsListProps? prev)
    {
        if(prev.HasValue && RishUtils.Compare(prev.Value.ids, Props.ids)) return;

        SetupChildren();
    }
    void IPropsListener<ItemsListProps>.PropsWillChange() { }

    protected override Element Render() => Col.Create(
        className: "items-list",
        children: new Children
        {
            H4.Create(text: Props.title),
            State.items
        });

    // This is called only when needed
    private void SetupChildre() {
        using(ManagedContext.New())
        {
            var items = new Children();
            foreach (var id in Props.ids)
            {
                if (items.Count > 0)
                {
                    children.Add(Separator.Create(key: (ulong)items.Count));
                }
                items.Add(Item.Create(key: (ulong)id, index: id));
            }
            SetItems(items);
        }
    }
}
[RishValueType]
public struct ItemsListProps
{
    public RishString title;
    public RishList<int> ids;
}
[RishValueType]
public struct ItemsListState
{
    public Children items;
}
{% endhighlight %}

### Use Sappy
In C#, every time you pass a method group (e.g., `action: OnClick`) as a lambda, a new delegate instance is allocated. This creates garbage. We have tested this in Editor, Mono builds and IL2CPP builds. It's unavoidable.

Rish integrates with [Sappy](https://github.com/clockworklabs/sappy) to solve this. Sappy generates cached delegates for your methods. You should use it. Especially in elements that get rendered or mounted/unmounted very frequently. We encourage you to [learn how Sappy works](https://github.com/clockworklabs/sappy).

Rishenerator automatically generates `SappyProps` (for callbacks) and `SappyState` (for setters) accessors for you.

{% highlight csharp %}
public partial class CachedLambdasExample : RishElement<CachedLambdasExampleProps>, IMountingListener
{
    void IMountingListener.ElementDidMount()
    {
        // ✅ Every time the element is mounted, we are reusing the same instance generated by Rishenerator.
        VolumeLevel.OnChange += SappyState.SetVolume;
    }
    void IMountingListener.ElementWillUnmount()
    {
        // ✅ Every time the element is unmounted, we are reusing the same instance generated by Rishenerator.
        VolumeLevel.OnChange -= SappyState.SetVolume;
    }
    protected override Element Render() => Row.Create(
        children: new Children
        {
            // ...
            // Every time the element is rendered, we are reusing the same instance generated by Rishenerator.
            Button.Create(action: SappyProps.Action),
        });
}
{% endhighlight %}

## Architecture and Logic
### Callbacks in Props
Callbacks (Delegates) should never affect the visual look of an element. Therefore, Rish ignores delegates when comparing Props to check if an element is dirty.

This can cause bugs if you pass a prop callback directly to a child element.

#### Scenario
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

We have a `PingPong` app that should ping pong a counter between 0 and 10. `Button` visual properties (color and label) always stay the same, but `action` changes.

1. Counter starts at 0.
2. User presses the button. Counter increases by 1. `counter` is now 1.
3. User presses the button. Counter increases by 1. `counter` is now 2.
...
11. User presses the button. Counter increases by 1. `counter` is now 10. `pong` is set to `true`.
12. `Button`'s color and label stay the same (action is ignored in the comparison), so it's not re-rendered. `AbstractButton`'s `action` is still pointing to `IncreaseCounter`.
13. User presses the button. Counter increases by 1. `counter` is now 11.

#### The Fix
Do not pass the raw callback from Props to the child. Instead, pass down a member method that invokes the delegate.

{% highlight csharp %}
public partial class Button : RishElement<ButtonProps>
{
    protected override Element Render() => AbstractButton.Create(
        action: SappyProps.Action, // Action method is auto-generated by Rishenerator.
        content: Div.Create(
            className: "button",
            style: new Style {
                backgroundColor = Props.color
            },
            children: H5.Create(text: Props.label)));

// ↓↓↓ Autogenerated by Rishenerator ↓↓↓
    [SapTarget]
    private void Action() => Props.action?.Invoke();
// ↑↑↑ Rishenerator ↑↑↑
}

[RishValueType]
public struct ButtonProps
{
    public Color color;
    public RishString label;
    public Action action;
}
{% endhighlight %}

This works because even though the comparison check for `ButtonProps` returns true, Rish still updates the Props value - it just doesn't flag the element as dirty.

### Divide and Conquer
If your `Render` method is becoming too big, split it up.

Symptoms of a bloated element:
- Deep nesting in `Render`.
- The urge to create helper methods that return an `Element`.

{% highlight csharp %}
public partial class InventoryWindow : RishElement<InventoryWindowProps, InventoryWindowState>
{
    protected override Element Render() => AbstractWindow.Create(
        header: Div.Create(
            children: new Children
            {
                Image.Create(/* ... */), // Background Image
                H5.Create(text: "Inventory"),
                AbstractButton.Create(/* ... */) // Close Button
            }),
        content: Col.Create(
            children: new Children
            {
                GetGrid(),
                Row.Create(
                    children: new Children
                    {
                        H6.Create(text: "Wallet"),
                        Image.Create(/* ... */), // Coins Sprite
                        P.Create(text: $"{State.coins}")
                    })
            }));

    [RequiresManagedContext]
    private Element GetGrid()
    {
        var rows = new Children();
        for (var i = 0; i < State.pockets; i += 4)
        {
            var children = new Children();
            for (var j = i; j < 4; j++)
            {
                children.Add(ItemFrame.Create(id: id, quantity: quantity));
            }
            rows.Add(children: children);
        }

        return Col.Create(children: rows);
    }
}
{% endhighlight %}

The easy solution is to create smaller, single purpose `RishElements`. This results in
- Faster Equality Checks: Shallow trees check faster.
- Granular Updates: If only one small part of the UI changes, only that small Element re-renders.

{% highlight csharp %}
public partial class InventoryWindow : RishElement<InventoryWindowProps, InventoryWindowState>
{
    protected override Element Render() => GameWindow.Create(
        title: "Inventory",
        content: Col.Create(
            children: new Children
            {
                ItemsGrid.Create(data: State.pockets),
                Wallet.Create(count: State.coins)
            }));
}
{% endhighlight %}

### Props and State
- **Fields:** Use public fields for raw data.
- **Properties:** Use properties for _inferred_ data only.
- **Methods:** Use methods for more expensive inferred data (>O(1)).
- **Reference Type Fields:** Avoid them like the plague. Reference types break determinism because they can mutate without Rish knowing.
- **Strings:** Use `RishString` instead of `string`.
    - It's a value type. It works with `RishList` (`RishString<string>` is invalid).
    - It provides an `IsEmpty` property (it uses `string.IsNullOrWhiteSpace` internally).
    - `null`, empty and white space strings all evaluate to `true` when compared to each other.
    - Length returns 0 for `null` or whitespace strings and we can avoid a lot of `null` checks (and `NullReferencePointerExceptions`).
    - Indexers and `Remove` method return `string.Empty` instead of `null` so we can avoid a lot of `null` checks (and `NullReferencePointerExceptions`).

### Styling
When it comes to best practices styling VisualElements, UI Toolkit is king. You should follow their recommended best practices (like favoring USS class names over inline styling).

## Style Guide And Conventions
### Element Classification
- **Foundational:** Generic, reusable. Zero game specific logic. Zero (or close to zero) opinions on styling. Roots' `AbstractButton`, `DragArea` or `Tooltip` are good examples.
- **Game Elements:** Specific to your project's style. If your game has elements like `SmallButton`, `LargeButton`, `Toggle` or all of Roots' Bootstrap elements are good examples.
- **Views:** Logical compositions of elements (e.g., `HUD`, `InventoryWindow`).

### Interfaces Implementations
Interfaces are implemented explicitly.

{% highlight csharp %}
void IMountingListener.ElementDidMount() { }
void IMountingListener.ElementWillUnmount() { }
{% endhighlight %}

You should not call any of these lifecycle methods manually and implementing them explicitly adds extra friction to prevent you from calling them by accident.

### Naming Conventions
- Props are called `[ElementName]Props`.
- State are called `[ElementName]State`.
- All fields in Props and State are `camelCase`.
    - It's the naming convention used in most of UIToolkit (for example all of `Style` properties) and we want to be consistent. 
    - They'll be used in `Create` methods and method arguments should be `camelCase`.

### Explicit Arguments
Arguments in `Create` methods are explicitly named. // EXPAND

### Nested Elements
Child elements that will only be created within the scope of a parent element are nested classes.

Small Elements with no custom Props or State are defined in the same file as the parent.

{% highlight csharp %}
public partial class NestedElementsExample : RishElement
{
    protected override Element Render() => Col.Create(
        children: new Children
        {
            H3.Create(text: "Title"),
            Nested.Create()
        });

    public partial class Nested : RishElement
    {
        protected override Element Render() => P.Create(text: "Body");
    }
}
{% endhighlight %}

More complex Elements are defined in separate files.

<figure>
    <figcaption>File 1</figcaption>
{% highlight csharp %}
public partial class NestedElementsExample : RishElement<NestedElementsExampleProps>
{
    protected override Element Render() => Col.Create(
        children: new Children
        {
            H3.Create(text: "Title"),
            Nested.Create(id: Props.id)
        });
}

[RishValueType]
public struct NestedElementsExampleProps {
    // ...
}
{% endhighlight %}
</figure>

<figure>
    <figcaption>File 2</figcaption>
{% highlight csharp %}
public partial class NestedElementsExample
{
    // ...

    public partial class Nested : RishElement<NestedProps, NestedState>
    {
        protected override Element Render() => Col.Create(
            children: new Children
            {
                // ...
            }
        );
    }

    [RishValueType]
    public struct NestedProps {
        public int id;
    }

    [RishValueType]
    public struct NestedState {
        // ...
    }
}
{% endhighlight %}
</figure>

### Readability
To maintain readability in declarative code, we enforce specific line-break rules.

#### Lists (`Children`, `ClassName`, `RishList`)
- **Single Item:** Inline.
- **Multiple Items:** One item per line.

{% highlight csharp %}
✅ Single
Div.Create(children: P.Create("Hello"));

✅ Multiple
Div.Create(
    children: new Children {
        P.Create(text: "Item 1"),
        P.Create(text: "Item 2")
    });
{% endhighlight %}

#### Create Methods
- **Short:** One line if ≤ 4 arguments and no multi-line lists.
- **Long:** One argument per line

{% highlight csharp %}
✅ Short
Foo.Create(name: "foo", visible: true, counter: 3);

✅ Long
Foo.Create(
    name: "foo",
    className: "my-class",
    counter: 3,
    visible: true,
    children: P.Create("Text")
);

✅ Long (because of children)
Foo.Create(
    className: "my-class",
    counter: 3,
    children: new Children {
        H5.Create("Title"),
        P.Create("Body")
    }
);
{% endhighlight %}