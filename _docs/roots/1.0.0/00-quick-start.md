---
title: Quick Start
slug: quick-start
sections:
  - Installation
  - Setup
  - Loading Assets
  - Responsive Style Sheets
  - Utilities
  - Samples
icon: download
---

Roots is a UI toolkit built on top of [Rish](/docs/rish/quick-start). Rish is very thin and provides virtually no elements out of the box. Roots provides actual building blocks to get your application running immediately.

#### What’s Included?
- **Layout Components:** Robust, responsive solutions for structural UI.
- **Foundational Elements:** Low-level, abstract building blocks.
- **Utilities:** Helper classes and methods to streamline development.
- **High-level Elements:** Optional reference elements to use as a starting point.

<div class="callout-block callout-block-note">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Note</h4>
        <p>Roots takes inspiration from established frameworks like Bootstrap but introduces custom approaches for Unity-specific challenges, such as tooltips and dropdowns.</p>
    </div>
</div>

## Installation
Installing Roots is simple. You can add the package via the Unity Package Manager using the Git URL, or by modifying your `manifest.json` file directly.

Add the following package URL: `https://github.com/clockworklabs/roots#[target-version]`.

### Dependencies
Roots requires the following dependencies to function correctly:
- [Rish](/docs/rish/quick-start): `io.clockworklabs.rish` (version `3.0.0+`).
- [Motion](https://github.com/clockworklabs/motion): `io.clockworklabs.motion` (version `1.7.9+`).

## Setup
To initialize the Roots ecosystem, add the following to the GameObject containing your `RishRoot`:
1. **`RootsSetup`:** Required for `ResponsiveStyleSheets`.
2. **`AssetsLoader`:** An abstract bridge to your asset pipeline.
  - Roots includes a `ResourcesLoader` for quick prototyping, but you should implement a custom version for production pipelines (Addressables, etc.).

If you plan to use animated elements (like `MotionDiv`), you also need a `MotionAutoUpdate` component in your project. You can skip this if you prefer to manually call `DoMotion.Step`.

## Responsive Style Sheets
Roots introduces a system similar to CSS `min-width` media queries, allowing you to stack USS files based on screen width.

1. **Create:** Go to `Create -> Roots -> ResponsiveStyleSheet`.
2. **Configure:** Add pairs of <strong>Min Width</strong> (`int`) and <strong>Style Sheet</strong> (`StyleSheet`).
  - `min-width <= 0`: The stylesheet is always loaded.
  - `min-width > 0`: Loaded only when screen width ≥ value.
3. **Include:** Add the `ResponsiveStyleSheet` to your `RootsSetup` component. 

Higher `min-width` sheets should be used to override styles defined in the "base" (0) or lower `min-width` sheets.

<figure>
    <figcaption>styles-xs.uss (min-width: 0)</figcaption>
{% highlight css %}
.example {
    margin: 8px;
}
{% endhighlight %}
</figure>

<figure>
    <figcaption>styles-lg.uss (min-width: 1024)</figcaption>
{% highlight css %}
.example {
    margin: 30px;
}
{% endhighlight %}
</figure>

`ResponsiveStyleSheets` are also convenient to pack style sheets together, even if you don't need responsive logic. You can just leave all the `min-size` as 0.

<div class="callout-block callout-block-danger">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Style Overrides</h4>
        <p>Roots appends these style sheets on top of those handled by <code>RishRoot</code>. Ensure your selectors account for this override order.</p>
    </div>
</div>

## Utilities
### Resolved Language Direction
`VisualElements` have a `languageDirection` property of type `public enum LanguageDirection{ Inherit, LTR, RTL }`. But when the value is `Inherit`, we don't know what the language direction is. Roots adds the `GetResolvedLanguageDirection` extension method which resolves the inherited directionlity.

### Style
Roots provides chain extension methods to set up inline `style`.

{% highlight csharp %}
Div.Create(
    style: StyleUtilites
        .FlexRow().
        .AlignItemsCenter()
        .Padding(16, 8)
    children: H3.Create(text: "Hello World"));
{% endhighlight %}

As always, the rule of favoring USS class names and style sheets over inline styling still apply, but these chain methods sometimes come handy.

### Vector Swizzling
We found ourselves needing to swizzle vector components all the time. So we created a bunch of extension methods just for that purpose and pack it as part of Roots.

{% highlight csharp %}
var vec0 = new Vector4(1, 2, 3, 4); // (1, 2, 3, 4)
var vec1 = vec0.sXY(); // (1, 2)
var vec2 = vec0.sX0Y(); // (1, 0, 2)
var vec3 = vec0.sZWWX(); // (3, 4, 4, 1)
{% endhighlight %}

## Samples
Roots comes with samples showing a wide range of UI Elements (from simple buttons to complex scroll views or responsive layouts).

1. Open the **Package Manager**.
2. Select the **Roots** package.
3. Go to the **Samples** tab and import **Rootstrap** and **Samples**.
4. Open the newly imported `Samples` scene and enter Play Mode.

Each sample is contained in a resizable container and has a "View Code" button to easily explore the code.