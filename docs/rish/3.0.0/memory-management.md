---
layout: docs
title: Memory Management
sections:
  - The Pooling System
  - Managed Contexts
  - Custom Managed Types
order: 6
icon: memory
---

Although we can't avoid all garbage, Rish is designed with a **Zero Garbage** philosophy.

In standard C# UI development, thousands of temporary objects (like lists) could be created every frame triggering the Garbage Collector (GC) and causing frame spikes and stuttering.

Rish solves this by implementing its own internal memory management system.

## The Pooling System
Rish automatically pools all reference types it handles.
- **Allocation:** When a new instance is needed, Rish retrieves one from a pre-allocated pool.
- **Deallocation:** When an instance is no longer needed (e.g., an element is unmounted), Rish automatically returns it to the pool.

<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title">Crucial</h4>
        <p>You should <strong>never</strong> call a constructor for a UI Element (or any reference type managed by Rish) directly. Always use the static <code>Create</code> methods. This ensures the element is correctly retrieved from the pool and tracked by Rish.</p>
    </div>
</div>

## The Pointer System
To efficiently handle these pooled resources without exposing the heavy reference types to the user, Rish uses a **Pointer System**.

Rish splits every major data structure into two parts:
- **The Pointer (Value Type):** A lightweight struct that you use in your code. It simply "points" to an index in the pool.
- **The Managed Object (Reference Type):** The heavy class instance that lives in the pool and holds the actual data.

Rish provides four core Pointer types:

<div class="table-responsive my-4">
    <table class="table table-striped">
        <thead>
            <tr>
                <th scope="col">Pointer Type</th>
                <th scope="col">Managed Type</th>
                <th scope="col">Description</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td><code>Element</code></td>
                <td><code>ManagedElement</code></td>
                <td>An element definition. It holds all the data that describes a UI element (type, props, children, ...).</td>
            </tr>
            <tr>
                <td><code>Children</code></td>
                <td><code>ManagedChildren</code></td>
                <td>An ordered list of <code>Element</code> pointers.</td>
            </tr>
            <tr>
                <td><code>RishList</code></td>
                <td><code>ManagedRishList</code></td>
                <td>A generic ordered list of value types.</td>
            </tr>
            <tr>
                <td><code>ClassName</code></td>
                <td><code>ManagedClassName</code></td>
                <td>An ordered list of string class names.</td>
            </tr>
        </tbody>
    </table>
</div>

## Managed Contexts
Because Rish is reusing memory, it needs to know exactly when a pointer is valid and when the underlying object can be returned to the pool. To track this, these pointers can _only_ be created within a **Managed Context**.

A `ManagedContext` tracks every resource you borrow from the pool during a specific scope.

##### The `[RequiresManagedContext]` Attribute
You might wonder why you can create elements freely inside your `Render` method without worrying about this.

This is because the Render method is flagged with the `[RequiresManagedContext]` attribute.

{% highlight csharp %}
namespace RishUI
{
    public abstract class RishElement<P> : IRishElement where P : struct
    {
        // ...
        [RequiresManagedContext]
        protected abstract Element Render();
        // ...
    }
}
{% endhighlight %}

Rish automatically opens a Context before calling `Render` and closes it afterward. If no one claimed interest on the Context, it can be freed (and all the references it owns return to the pool). If somebody claimed interest, it will stay around for as long as there is interest.

##### Manual Contexts
If you try to create a Pointer type outside of a valid context (e.g., in a method to update state), you will get a compilation error.

To fix this, you must manually open a context using `ManagedContext.New()`:

{% highlight csharp %}
public void UpdateState() {
    // ❌ Error: Cannot create Rish types outside a Managed Context
    var failedElement = P.Create(text: "Test"); // This throws a compilation error
    var failedList = new RishList(); // This throws a compilation error

    // ✅ Correct: Wrapped in a context
    using(ManagedContext.New()) {
        var element = P.Create(text: "Test"); // This works
        var list = new RishList(); // This works

        SetElement(element);
        SetList(list);
    }
    // At this closing brace, 'element' and 'list' are "closed" and their Managed Types can't be modified anymore.
}
{% endhighlight %}

## Custom Managed Types
You can leverage Rish's high-performance pooling system for your own data structures.

To create a custom pooled type:
1. **The Reference:** Create a class that implements `IManaged` (must have a parameterless constructor).
2. **The Pointer:** Create a struct that implements `IReference`.

Once implemented, Rish will automatically pool instances of your class and allow you to use them safely within Managed Contexts, just like native Rish types.

<div class="alert alert-info" role="alert">
    A more in depth guide coming soon.
</div>