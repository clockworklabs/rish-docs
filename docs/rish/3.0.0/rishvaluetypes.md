---
layout: docs
title: RishValueTypes
sections:
  - Members
  - Comparison
  - References
order: 5
---

Intro.

## Members
You can define all the members you need/want. We recommend:
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

## Comparison
Rish needs to compare RishValueTypes (for example, compare Props to determine if a RishElement is dirty and should be re-rendered or not). Rishenerator (which we highly suggest using) auto-generates all the necessary source code to achieve this. You can provide attributes to specify how to compare each field:
- `IgnoreComparison`: To ignore fields when comparing.
- `EqualityOperatorComparison`: To use `==` when comparing.
- `EqualsMethodComparison`: To compare using the `Equals` method.
- `EpsilonComparison`: Only for `float` fields, to compare using Unity's `Mathf.Approximately` function.
If no comparison attribute is provided, the default behavior is as follows:
- For reference type fields:
    - Delegate types: To ignore them.
    - Other reference types: To compare references.
- For value types: To use a Comparer, if defined, or to perform a low-level binary comparison in every other case.

Rishenerator's auto generated comparisons will only compare fields. This is one of the reasons we recommend using properties only for inferred data. 

### Comparers
Some special types might need to handle comparisons in a more manual way. In this cases, you can implement a static predicate `(T, T) -> Bool` function flagged with the `Comparer` attribute. If Rish finds one of these defined, it will use it to compare value type fields.

For example, Rish's `Element` is a value type that simply holds an id for an internally managed reference type. Two element definitions could be equal to each other even if their ids are different, so we defined the following Comparer to use the Equals function of the internal reference type instead:
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