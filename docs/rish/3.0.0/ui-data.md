---
layout: docs
title: UI Data
sections:
  - Value Types
  - Reference Types
order: 5
icon: database
---

We want to store in Props and State all the relevant data that each element needs to render and accomplish a deterministic UI. 

## Value Types
In Rish we highly encourage using value types whenever possible as much as possible. Props and State are enforced to be structs and we don't recommend using any reference types for data in Props and State because they could change internally without triggering an element to re-render. Rish offers many useful value types like `Element`, `Children`, `RishString` and `RishList`.

When creating a new value type to hold data, it's recommended to add the `RishValueType` attribute for Rishenerator to auto generate important code for memory management and performance. If you're using Rishenerator (which again, we can't recommend enough), all Props and State will be enforced to have this attribute.

### Members
You can define all the members you want and need. We recommend following these guidelines:
- fields for raw data that is passed down;
- properties for inferred data;
- methods for more expensive (>O(1)) inferred data;
- avoid reference types as much as possible (to help guarantee UI determinism).

For example:

{% highlight csharp %}
[RishValueType]
public struct FooProps {
    public bool flag0;
    public bool flag1;
    public uint n;

    public bool flag => flag0 || flag1;

    public ulong GetFactorial() {
        ulong result = 1;
        for (var i = 1; i <= n; i++)
        {
            result *= i;
        }
        return result;
    }
}
{% endhighlight %}

### Comparison
Rish needs to compare value types (for example, compare Props to determine if an element is dirty and should be re-rendered or not). Rishenerator auto-generates all the necessary source code to achieve this easily. You can provide attributes to specify how to compare each field:
- `IgnoreComparison`: To ignore fields when comparing.
- `EqualityOperatorComparison`: To use `==` when comparing.
- `EqualsMethodComparison`: To compare using the `Equals` method.
- `EpsilonComparison`: Only for `float` fields, to compare using Unity's `Mathf.Approximately` function.

If no comparison attribute is provided, the default behavior is as follows:
- For reference type fields:
    - Delegate types: To ignore them.
    - Other reference types: To compare references.
- For value types: To use a Comparer if defined, or to perform a low-level binary comparison if no Comparar is found.

Rishenerator's auto generated code will only compare fields (and ignore all properties, which we recommend leaving for inferred data). 

#### Comparers
Ideally, for speed, we can perform a low-level binary memory comparison to compare value types but some times it's not possible (for example if a field needs to be ignored or compared using the Equals method or another Comparer) and Rishenerator will auto generate a Comparer function.

In some extra special cases, you might even need to handle comparisons in a more manual way. In these cases, you can implement a static predicate `(T, T) -> Bool` function flagged with the `Comparer` attribute.

For example, Rish's `Element` is a value type that simply holds an id for an internally managed reference type. Two element definitions could be equal to each other (define the same UI element) even if their ids are different, so we defined the following manual Comparer to use the Equals function of the internal reference type instead:
{% highlight csharp %}
[Comparer]
private static bool Equals(Element a, Element b)
{
    var aSet = a.Valid;
    var bSet = b.Valid;
    if (aSet ^ bSet)
    {
        return false;
    }
    if (!aSet)
    {
        return true;
    }
    
    var aDefinition = a.GetDefinition();
    var bDefinition = b.GetDefinition();

    var aDisposed = aDefinition == null;
    var bDisposed = bDefinition == null;
    if (aDisposed || bDisposed)
    {
        return false;
    }

    return aDefinition.Equals(bDefinition);
}
{% endhighlight %}

## Reference Types
Once again, we recommend not using reference types to hold UI data. The usual approach is to subscribe to events and store in State (in value types) the data we care about in the Render function.

{% highlight csharp %}
public partial class InventoryPocketTitle : RishElement, IMountingListener, IPropsListener {
    void IMountingListener.ComponentDidMount() {
        GameState.PlayerInventory.OnChange += Setup;
    }
    void IMountingListener.ComponentWillUnmount() {
        GameState.PlayerInventory.OnChange -= Setup;
    }

    void IPropsListener.PropsDidChange() {
        Setup(GameState.PlayerInventory);
    }
    void IPropsListener.PropsWillChange() { }

    protected override Element Render() => ItemFrame.Create(id: State.itemId, quantity: State.quantity);

    private void Setup(PlayerInventory inventory) {
        var pocket = inventory.Get(Props.index);
        SetItemId(pocket.ItemId);
        SetQuantity(pocket.Quantity);
    }
}

[RishValueType]
public struct InventoryPocketProps
{
    public int index;
}

[RishValueType]
public struct InventoryPocketState
{
    public int itemId;
    public int quantity;
}
{% endhighlight %}

But of course in some cases you may need to use reference types. If you know the type won't mutate, even though it's discouraged, you may just use it directly in your Props or State and remember that, by default, they'll be compared by reference.

For more complex scenarios, Rish also provides an API that we cover in the next section.