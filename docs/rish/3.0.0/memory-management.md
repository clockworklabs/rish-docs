---
layout: docs
title: Memory Management
sections:
  - Managed Contexts
  - Custom Managed Types
order: 6
icon: memory
---

Rish is internally handling a lot of reference types to keep things running smoothly and fast. Our goal is to not create any garbage on Rish's side.

Rish automatically pools all reference types it handles. When a new instance is needed, Rish gets one from the pool or creates a new one if the pool is empty. To know when a reference type is not needed anymore, Rish implements its own memory management system. Similarly to a garbage collector, it collects all the instances that are not needed anymore and puts them back into their respective pool.

All UI Elements (VisualElements and RishElements) are pooled and handled by Rish. You should never call a constructor for any UI Element, instead you should use the `Create` methods to create element definitions that Rish will in turn use to get and setup the right UI Element.

## Managed Contexts
Some value types act as references or pointers to instances of a reference type. Internally, Rish handles and reuses the reference type; externally, a value type is used and passed around. Rish has four of these types: `Element`, `Children`, `RishList` and `ClassName`.
- `Element`: A reference to `ManagedElement`, an element definition holding all the necessary information to describe an UI element (type, props, children, ...).
- `Children`: A reference to `ManagedChildren`, an ordered list of `Element`s.
- `RishList`: A reference to `ManagedRishList`, a generic ordered list of value type elements.
- `ClassName`: A reference to `ManagedClassName`, an ordered list of string class names.

Rish needs to know if any of these are being used and keep them "alive" while they are needed. To accomplish this, these types can only be created within the scope of a `ManagedContext`.

{% highlight csharp %}
public void Foo() {
    var failedElement = P.Create(text: "Test"); // This throws a compilation error
    var failedList = new RishList(); // This throws a compilation error

    using(ManagedContext.New()) {
        var element = P.Create(text: "Test"); // This works
        var list = new RishList(); // This works
    }
}
{% endhighlight %}

In this example, we would would say that `element` and `list` are "owned" by the surrounding `ManagedContext`. Rish will keep a `ManagedContext` (and all the references it owns) around for as long as someone claims it. Rish automatically claims all the necessary `ManagedContext`s every time we set Props or State, and every time they change or a UI Elements is unmounted, the previously claimed `ManagedContext`s are released (and the references it owns freed back to the pool).

The reason why you can call a `Create` method or create a `RishList` inside a `Render` function is because the abstract `Render` function definition has the `RequiresManagedContext` attribute

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

which forces the `Render` function to be called within the scope of a `ManagedContext`.

## Custom Managed Types
You can use the same API that `Element` or `RishList` use to create your own reference types managed by Rish.

Your reference type must implement the `IManaged` interface and must have a public parameterless constructor. Your value type must implement the `IReference` interface.

And that's it, now Rish will handle your new type.
