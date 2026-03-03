---
title: Layouting
slug: layouting
sections:
    - Linear Layouts
    - Responsive Layouts
icon: table-columns
---

## Linear Layouts
The `Stack` element is a foundational layout RishElement. It acts as a generic container that arranges elements in a single direction.

### Props
- `Stack.Direction direction`: `Vertical` (default) or `Horizontal`.
- `bool reverse`: If true, children are arranged in reverse order.
- `float gap`: Pixel spacing between children.
- `Element separator`: An element (like a divider) rendered between each child.
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `Children children`: The elements to be arranged.

### Col and Row
To keep code clean, Roots provides two wrappers that pre-set the direction:
- `Col`: A `Stack` with `Vertical` direction.
- `Row`: A `Stack` with `Horizontal` direction.

## Responsive Layouts
While `Stacks` may cover 90% of use cases, Roots also includes a responsive system inspired by modern web frameworks.

### Breakpoints
Breakpoints are width thresholds that define layout behavior. Roots uses six standard tiers:

<div class="table-responsive my-4">
    <table class="table table-striped">
        <thead>
            <tr>
                <th scope="col">Breakpoint</th>
                <th scope="col">Key</th>
                <th scope="col">Default Min Width</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>Extra Small</td>
                <td><code>xs</code></td>
                <td><i>Always</i> 0.</td>
            </tr>
            <tr>
                <td>Small</td>
                <td><code>sm</code></td>
                <td>576</td>
            </tr>
            <tr>
                <td>Medium</td>
                <td><code>md</code></td>
                <td>768</td>
            </tr>
            <tr>
                <td>Large</td>
                <td><code>lg</code></td>
                <td>1024</td>
            </tr>
            <tr>
                <td>Extra Large</td>
                <td><code>xl</code></td>
                <td>1365</td>
            </tr>
            <tr>
                <td>Extra Extra Large</td>
                <td><code>xxl</code></td>
                <td>1820</td>
            </tr>
        </tbody>
    </table>
</div>

<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>The 4/3 Rule</h4>
        <p>By default, Roots calculates breakpoints using a 4/3 ratio. If you define <code>sm</code>, the system will automatically calculate the other breakpoints. If you need custom ratios, define all breakpoints manually.</p>
    </div>
</div>

### Responsive Context
A `ResponsiveContext` is the root of all responsive layouts. Elements react to their parent `ResponsiveContext`.

You can have multiple `ResponsiveContexts` (e.g. resizable windows, viewports) and elements inside them will adapt to the specific width of each `ResponsiveContext` independently.

You define your responsive breakpoints through `ResponsiveContext`'s Props.

#### Props
- `int sm`: Min width for Small breakpoint. Extra Small min width is _always_ 0.
- `int md`: Min width for Medium breakpoint.
- `int lg`: Min width for Large breakpoint.
- `int xl`: Min width for Extra Large breakpoint.
- `int xxl`: Min width for Extra Extra Large breakpoint.
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `Children children`: Container's children.
- `Action<float, ResponsiveBreakpoint> onResize`: Callback that gets called when the container gets resized.

### Containers
Containers are the most basic responsive layout element. They are used to contain, pad, and (sometimes) center the content within them.
- `Container`: Stays 100% wide until its specific breakpoint is hit, then applies a `max-width`. Equivalent to `.container-{breakpoint}` in Bootstrap.
- `FluidContainer`: Always spans 100% of the available width. Equivalent to `.fluid-container` in Bootstrap.

The table below illustrates how each container compares to each other at each breakpoint (assuming default breakpoint values).

<div class="table-responsive my-4">
    <table class="table table-striped">
        <thead>
            <tr>
                <th scope="col"></th>
                <th scope="col">
                    xs (≥ 0)
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    sm (≥ 576)
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    md (≥ 768)
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    lg (≥ 1024)
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    xl (≥ 1365)
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    xxl (≥ 1820)
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>
                    <code>Container</code>
                    <br>
                    <small class="opacity-25">Extra Small</small>
                </td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 576px</td>
                <td class="font-monospace">100% / 768px</td>
                <td class="font-monospace">100% / 1024px</td>
                <td class="font-monospace">100% / 1365px</td>
                <td class="font-monospace">100% / 1820px</td>
            </tr>
            <tr>
                <td>
                    <code>Container</code>
                    <br>
                    <small class="opacity-25">Small</small>
                </td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 576px</td>
                <td class="font-monospace">100% / 768px</td>
                <td class="font-monospace">100% / 1024px</td>
                <td class="font-monospace">100% / 1365px</td>
                <td class="font-monospace">100% / 1820px</td>
            </tr>
            <tr>
                <td>
                    <code>Container</code>
                    <br>
                    <small class="opacity-25">Medium</small>
                </td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 768px</td>
                <td class="font-monospace">100% / 1024px</td>
                <td class="font-monospace">100% / 1365px</td>
                <td class="font-monospace">100% / 1820px</td>
            </tr>
            <tr>
                <td>
                    <code>Container</code>
                    <br>
                    <small class="opacity-25">Large</small>
                </td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 1024px</td>
                <td class="font-monospace">100% / 1365px</td>
                <td class="font-monospace">100% / 1820px</td>
            </tr>
            <tr>
                <td>
                    <code>Container</code>
                    <br>
                    <small class="opacity-25">Extra Large</small>
                </td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 1365px</td>
                <td class="font-monospace">100% / 1820px</td>
            </tr>
            <tr>
                <td>
                    <code>Container</code>
                    <br>
                    <small class="opacity-25">Extra Extra Large</small>
                </td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 100%</td>
                <td class="font-monospace">100% / 1820px</td>
            </tr>
            <tr>
                <td>
                    <code>FluidContainer</code>
                </td>
                <td class="font-monospace">100% / Not Set</td>
                <td class="font-monospace">100% / Not Set</td>
                <td class="font-monospace">100% / Not Set</td>
                <td class="font-monospace">100% / Not Set</td>
                <td class="font-monospace">100% / Not Set</td>
                <td class="font-monospace">100% / Not Set</td>
            </tr>
        </tbody>
    </table>
</div>

#### `Container` Props
- `ResponsiveBreakpoint breakpoint`: Breakpoint.
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `Element content`: Container's content.

#### `FluidContainer` Props
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `Element content`: Container's content.

### Grid
Roots provides a powerful responsive grid system for complex layouts. If you're familiar with Bootstrap's (or other similar CSS responsive frameworks) Grid, you will find Roots' solution familiar. Although it has some key differences.

It uses a Mobile-First Inheritance model: settings for `xs` flow up to `xxl`.

#### How it works
A `Grid` defines how many units wide it is per breakpoint. Grid sizes are inherited from lower breakpoints.

If columns exceed the Grid's unit count, they wrap to a new line. If no grid size is set, the Grid automatically fits all columns into one row.

Columns and rows are separated by a gutter. Grid gutters are also inherited from lower breakpoints.

#### Props
- `int? xs`: Grid size to use in Extra Small `ResponsiveContexts`. If not set, the grid will set the size to fit all `cols` in one row. 
- `int? sm`: Grid size in Small `ResponsiveContexts`. If not set, it inherits `xs` value.
- `int? md`: Grid size in Medium `ResponsiveContexts`. If not set, it inherits `sm` value.
- `int? lg`: Grid size in Large `ResponsiveContexts`. If not set, it inherits `md` value.
- `int? xl`: Grid size in Extra Large `ResponsiveContexts`. If not set, it inherits `lg` value.
- `int? xxl`: Grid size in Extra Extra Large `ResponsiveContexts`. If not set, it inherits `xl` value.
- `Gutter? xsGutter`: Gutter to use in Extra Small `ResponsiveContexts`. If not set, it's 0 by default. 
- `Gutter? smGutter`: Gutter to use in Small `ResponsiveContexts`. If not set, it inherits `xs` value.
- `Gutter? mdGutter`: Gutter to use in Medium `ResponsiveContexts`. If not set, it inherits `sm` value.
- `Gutter? lgGutter`: Gutter to use in Large `ResponsiveContexts`. If not set, it inherits `md` value.
- `Gutter? xlGutter`: Gutter to use in Extra Large `ResponsiveContexts`. If not set, it inherits `lg` value.
- `Gutter? xxlGutter`: Gutter to use in Extra Extra Large `ResponsiveContexts`. If not set, it inherits `xl` value.
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `RishList<ColData> cols`: Columns data.

#### `ColData`
Each column has a size in Grid Units per breakpoint. Column sizes are inherited from lower breakpoints.

If no column size is set, then it's assumed to be 1. A size of 0 is valid, which is useful when you need to hide a column only in certain breakpoints.

Children in a column will be separated by the `Grid`'s vertical gutter.

- `int? xs`: Column size (in Grid Units) in Extra Small `ResponsiveContexts`. If not set, it's assumed to be 1.
- `int? sm`: Column size (in Grid Units) in Small `ResponsiveContexts`. If not set, it inherits `xs` value.
- `int? md`: Column size (in Grid Units) in Medium `ResponsiveContexts`. If not set, it inherits `sm` value.
- `int? lg`: Column size (in Grid Units) in Large `ResponsiveContexts`. If not set, it inherits `md` value.
- `int? xl`: Column size (in Grid Units) in Extra Large `ResponsiveContexts`. If not set, it inherits `lg` value.
- `int? xxl`: Column size (in Grid Units) in Extra Extra Large `ResponsiveContexts`. If not set, it inherits `xl` value.
- `VisualAttributes descriptor`: Styling information.
- `Children children`: Children.

#### Examples
