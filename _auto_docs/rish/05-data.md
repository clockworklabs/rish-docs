---
title: UI Data
slug: data
sections:
  - Value Types
  - Reference Types
icon: database
---

In Rish, UI is a **pure function of data**. To guarantee a deterministic UI, Rish needs to know exactly when data changes.

## Value Types
We strictly enforce using **Value Types (structs)** for Props and State.
- **Safety:** Value types are copied by value. This prevents accidental mutation of data that might be shared across different parts of the UI tree.
- **Performance:** Structs avoid garbage collection allocation overhead.
- **Determinism:** Deep comparison of value types is reliable; comparing references is not.

Rish provides some useful optimized value types like `Element`, `Children` and `RishList`.

### The `[RishValueType]` Attribute
Adding `[RishValueType]` to your struct definitions triggers Rishenerator to auto-generate high-performance code for memory management and, _crucially_, Equality Comparisons.

Every Props and State struct must be flagged with `[RishValueType]`.

#### Guidelines
When defining your data structures:
- **Fields:** Use public fields for raw data.
- **Properties:** Use properties for _inferred_ data (calculated from fields).
- **Methods:** Use methods for more expensive calculations (>O(1)).

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

_Note: Rishenerator's auto-generated equality checks only look at Fields. Properties are ignored during dirty checking._

### Equality Checks
Rish determines if an element needs to re-render by comparing the **New Props** vs. the **Old Props** or the **New State** vs. the **Old State**. If they are not equal, the element is flagged **Dirty** and re-renders.

You can control how specific fields are compared using attributes:

<div class="table-responsive my-4">
    <table class="table table-striped">
        <thead>
            <tr>
                <th scope="col">Attribute</th>
                <th scope="col">Behavior</th>
                <th scope="col">Use Case</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>[IgnoreComparison]</code></td>
                <td>Field is ignored during checks.</td>
                <td>Data that doesn't affect the visual output.</td>
            </tr>
            <tr>
                <td><code>[EpsilonComparison]</code></td>
                <td>Uses <code>UnityEngine.Mathf.Approximately</code>.</td>
                <td>Floating point numbers (float).</td>
            </tr>
            <tr>
                <td><code>[EqualityOperatorComparison]</code></td>
                <td>Uses <code>==</code> operator.</td>
                <td>Types with custom operator overloads.</td>
            </tr>
            <tr>
                <td><code>[EqualsMethodComparison]</code></td>
                <td>Uses <code>.Equals()</code> method.</td>
                <td>Complex types.</td>
            </tr>
        </tbody>
    </table>
</div>

When no comparison attribute is provided, the default behavior is:
- Reference types: Reference equality.
    - Delegates: Ignored.
- Value Types: Low-level binary comparison (fastest) unless a Comparer is found.

#### Comparers
Rishenerator will produce Comparer methods whenever a fast memory comparison is not possible (for example, if a field should be ignored or should be compared using another Comparer).

In rare cases, you may need manual control over how a specific type is compared and you can define a static `(T, T) -> Bool` Custom Comparer method flagged with [Comparer].

For example, Rish's `Element` is a Pointer Value Type (and it just holds an id) and to compare Element Definitions we should call the `Equals` method of the internal reference type instead:

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
While we discourage using reference types in Props/State, you often need to interface with existing game systems.

**The Solution:** Do not put the mutable object itself in Props. Instead, subscribe to changes and copy the relevant data into your State.

{% highlight csharp %}
public partial class InventoryPocket : RishElement<InventoryPocketProps, InventoryPocketState>, IMountingListener, IPropsListener {
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

        // This triggers the re-render ONLY if the values of this pocket actually changed
        SetItemId(pocket.ItemId);
        SetQuantity(pocket.Quantity);
    }
}
{% endhighlight %}

##### Direct Reference Usage (Use with Caution)
If you must pass a reference type (e.g., a `Texture2D` or a `ScriptableObject` configuration), remember that Rish defaults to **Reference Equality** and if the content of the object changes but the reference stays the same, Rish will not detect the change and will not re-render.