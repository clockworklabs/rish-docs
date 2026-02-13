---
layout: docs
title: The Render Pipeline
sections:
  - Element Definitions
  - Mounting and Unmounting
  - The Update Cycle
  - Keys
  - Manual Dirty
order: 2
# icon: timeline
icon: arrows-turn-to-dots
---

Rish manages the complexity of updating and rendering the UI. It handles the synchronization between the Virtual Tree and the Visual Tree, automatically pooling elements to minimize garbage collection and optimizes memory usage.

## Element Definitions
It is important to understand the distinction between an element _instance_ and an element _definition_.
- The `Render` method returns an `Element`.
- `Element` is a Pointer (more on these in the [Memory Management guide](/docs/rish/3.0.0/memory-management)) to an Element Definition.
- An **Element Definition** is a blueprint for an Element. It holds the type of UI element to create, its property values, its styling information, and its children. It is not the live UI object itself.

## Mounting and Unmounting
The lifecycle of an Element begins when it is "mounted" (added to the tree).

### Mounting
When a `RishElement` is rendered for the first time:
1. Rish retrieves an instance of the required type (based on the `Element` returned in the `Render` method) from the Object Pool.
2. It mounts this instance as a child of the current parent.
3. **If it is a `RishElement`:** Rish initializes its Props and flags it as `Dirty`. This triggers this new Element to be rendered next, continuing the cycle.
4. **If it is a `VisualElement`:** Rish sets the styling properties (name, class, and inline style), calls `Setup` with the Props, and creates/mounts the children, continuing the cycle.

### Unmounting
When an element is no longer needed (e.g., due to a conditional check in your `Render` method):
1. Rish recursively unmounts all of the element's children first.
2. The element itself is unmounted.
3. The instances are reset and returned to the Object Pool, ready to be reused immediately.

## The Update Cycle
Elements trigger an **Update Chain** whenever they are flagged as `Dirty`.

An element is automatically flagged as dirty when:
- Its Props change (passed down from a parent).
- Its State changes (internal logic).

## REVISIT
When a `RishElement` is dirty, Rish calls its `Render` function. This process is recursive but efficient:
- If the Render call results in the exact same structure and properties, the update stops there.
- If the Props (or styling) change, the element updates.
- If the structure changes (elements added/removed), Rish reconciles the lists.
- **Optimization:** If a parent renders but a child's Props have not changed, Rish skips re-rendering that child entirely.
## REVISIT

#### Example
Let's look at how Rish handles updates with a concrete example.

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
            // Conditional rendering based on State
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

##### Scenario 1: Not Hovered (Initial State)
- **Result:** `Row` -> `H1` ("The element is:"), `P` ("not hovered..."), `P` ("yet.").
- **Action:** Rish mounts all these elements from the pool.

##### Scenario 2: User Hovers (State Change)
1. `OnHoverStart` modifies State. `DirtyExample` is flagged **Dirty**.
2. Rish calls `Render`.
3. **Result:** `Row` -> `H1` ("The element is:"), `P` ("hovered.").
4. **Reconciliation:**
  - `Row`: Reused.
  - `H1`: Reused (no changes).
  - `P`: Reused. Text property updated to "hovered.".
  - `P`: **Unmounted** and returned to pool.

##### Scenario 3: User Stops Hovering (State Change)
1. `OnHoverEnd` modifies State. `DirtyExample` is flagged **Dirty**.
2. Rish calls `Render`.
3. **Result:** `Row` -> `H1` ("The element is:"), `P` ("not hovered..."), `P` ("yet.").
4. **Reconciliation:**
  - `Row`: Reused.
  - `H1`: Reused (no changes).
  - `P`: Reused. Text property updated to "not hovered...".
  - `P`: **Mounted** with text property "yet.".

## Keys
Rish tries to reuse elements in an intelligent way and takes the order of elements into consideration. This works fine for static lists, but can cause issues when the order changes or items are inserted/removed dynamically.

If you have multiple children of the same type, Rish might reuse the wrong instance for the wrong data, resulting in inefficient updates.

{% highlight csharp %}
private partial class KeysExample : RishElement<NoProps, KeysExampleState>
{
    public KeysExampleState {
        RegisterCallback<HoverStartEvent>(OnHoverStart);
        RegisterCallback<HoverEndEvent>(OnHoverEnd);
    }

    protected override Element Render() => Col.Create(
        children: new Children {
            H1.Create(text: value: "Hello world."),
            State.hovered ? P.Create(text: "Element is being hovered.") : Element.Null, // Conditional rendering based on State
            P.Create(text: "Goodbye world.")
        });

    private void OnHoverStart(HoverStartEvent evt) => SetHover(true);
    private void OnHoverEnd(HoverEndEvent evt) => SetHover(false);
}

[RishValueType]
public struct KeysExampleState {
    public bool hovered;
}
{% endhighlight %}

##### Scenario 1: Not Hovered (Initial State)
- **Result:** `Col` -> `H1` ("Hello world."), `P` ("Goodbye world.").
- **Action:** Rish mounts all these elements from the pool.

##### Scenario 2: User Hovers (State Change)
1. `OnHoverStart` modifies State. `KeysExample` is flagged **Dirty**.
2. Rish calls `Render`.
3. **Result:** `Col` -> `H1` ("Hello world."), `P` ("Element is being hovered."), `P` ("Goodbye world.").
4. **Reconciliation:**
  - `Col`: Reused.
  - `H1`: Reused (no changes).
  - `P`: Reused. Text property updated to "Element is being hovered.".
  - `P`: **Mounted** with text property "Goodbye world".

##### Scenario 3: User Stops Hovering (State Change)
1. `OnHoverEnd` modifies State. `KeysExample` is flagged **Dirty**.
2. Rish calls `Render`.
3. **Result:** `Col` -> `H1` ("Hello world."), `P` ("Goodbye world.").
4. **Reconciliation:**
  - `Col`: Reused.
  - `H1`: Reused (no changes).
  - `P`: Reused. Text property updated to "Goodbye world.".
  - `P`: **Unmounted** and returned to pool.

We are clearly not being as efficient as we could be. We can prevent this using Keys.

A **Key** is a ulong identifier that tells Rish: _"This specific element definition belongs to this specific instance"_.

{% highlight csharp %}
private partial class KeysExample : RishElement<NoProps, KeysExampleState>
{
    public KeysExampleState {
        RegisterCallback<HoverStartEvent>(OnHoverStart);
        RegisterCallback<HoverEndEvent>(OnHoverEnd);
    }

    protected override Element Render() => Col.Create(
        children: new Children {
            H1.Create(text: value: "Hello world."),
            State.hovered ? P.Create(key: 2, text: "Element is being hovered.") : Element.Null, // Conditional rendering based on State
            P.Create(key: 1, text: "Goodbye world.")
        });

    private void OnHoverStart(HoverStartEvent evt) => SetHover(true);
    private void OnHoverEnd(HoverEndEvent evt) => SetHover(false);
}

[RishValueType]
public struct KeysExampleState {
    public bool hovered;
}
{% endhighlight %}

##### Scenario 1: Not Hovered (Initial State)
- **Result:** `Col` -> `H1` ("Hello world."), `P [1]` ("Goodbye world.").
- **Action:** Rish mounts all these elements from the pool.

##### Scenario 2: User Hovers (State Change)
1. `OnHoverStart` modifies State. `KeysExample` is flagged **Dirty**.
2. Rish calls `Render`.
3. **Result:** `Col` -> `H1` ("Hello world."), `P [2]` ("Element is being hovered."), `P [1]` ("Goodbye world.").
4. **Reconciliation:**
  - `Col`: Reused.
  - `H1`: Reused (no changes).
  - `P [2]`: **Mounted** with text property "Element is being hovered.".
  - `P [1]`: Reused (no changes)".

##### Scenario 3: User Stops Hovering (State Change)
1. `OnHoverEnd` modifies State. `KeysExample` is flagged **Dirty**.
2. Rish calls `Render`.
3. **Result:** `Col` -> `H1` ("Hello world."), `P` ("Goodbye world.").
4. **Reconciliation:**
  - `Col`: Reused.
  - `H1`: Reused (no changes).
  - `P [2]`: **Unmounted** and returned to pool.
  - `P [1]`: Reused (no changes)".

In this updated example, Rish uses the keys (`1` and `2`) to ensure that the `P` element with `key: 1` is always reused for the "Goodbye" message, and the `P` element with `key: 2` is specifically the one mounted and unmounted when the state changes.

<div class="callout-block callout-block-success">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Best practice</h4>
        <p>Always provide Keys when generating lists of elements in a loop or when swapping elements conditionally.</p>
    </div>
</div>

## Manual Dirty
In 99% of cases, Rish's automatic Props/State detection is sufficient. However, if you are wrapping non-Rish systems or using complex instance-based logic, you might need to trigger updates manually.

You can implement `IManualState` and use the `Dirty()` method.

{% highlight csharp %}
private partial class ManualDirtyExample : RishElement, IManualState
{
    // Instance state that Rish's struct-based State doesn't track automatically
    private HashSet<int> Indices { get; } = new();
    
    // Called when pulled from the pool
    void IManualState.Restart()
    {
       Indices.Clear();
    }
    
    protected override Element Render() => H5.Create(text: $"Hovered by {Indices.Count} pointers.");

    private void OnHoverStart(HoverStartEvent evt) => AddPointer(evt.pointerId);
    private void OnHoverEnd(HoverEndEvent evt) => RemovePointer(evt.pointerId);
    private void AddPointer(int id)
    {
        if(Indices.Add(id))
        {
            Dirty(); // Flags the element to re-render
        }
    }
    private void RemovePointer(int id)
    {
        if(Indices.Remove(id))
        {
            Dirty(); // Flags the element to re-render
        }
    }
}
{% endhighlight %}

### The `Dirty(bool immediate)` method:
- `Dirty(false)` (Default): Schedules the update. It may or may not happen on the current update step.
- `Dirty(true)`: Forces the update to happen during the current update step. Use this only when strictly necessary, as it can result in elements being rendered more than once in the update step.