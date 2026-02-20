---
title: Layouting
slug: layouting
sections:
    - Stack
    - Responsive Layouts
icon: table-columns
---

## Stack
The `Stack` element is a foundational layout RishElement. It acts as a generic container, wrapping around the elements to be arranged.

### Props
- `Stack.Direction direction`: The direction of the stack. `Vertical` by default.
- `bool reverse`: Whether or not the children should be arranged in reversed order.
- `float gap`: Gap in pixels between children.
- `DOMDescriptor descriptor`: Styling information. Expanded in `Create` method.
- `Children children`: All children to be arranged.
- `Element separator`: A separator to be added in between children.

### Col and Row
Roots provides two wrappers to skip passing down the direction.
- `Col` creates a `Stack` with Vertical direction.
- `Row` creates a `Stack` with Horizontal direction.

`Stacks` (`Col` and `Row`) might be all you need 90% of the time, but Roots provides more advanced and powerful options too.

## Responsive Layouts
### Breakpoints
Breakpoints are customizable widths that determine how your responsive layout behaves. They are the building blocks of responsive design.

There are six breakpoints that you can set via `ResponsiveContext` Props.

<div class="table-responsive my-4">
    <table class="table table-striped">
        <thead>
            <tr>
                <th scope="col">Breakpoint</th>
                <th scope="col">Short Name</th>
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

The default ratio between breakpoints is `4/3`. This means that if you set `md` to 900, `lg` will be 1200 (`900 * 4 / 3`), `xl` will be 1600 (`1200 * 4 / 3`) and `xxl` will be 2133 (`floor(1600 * 4 / 3)`). This behavior guarantess that breakpoints "make sense" between each other.

If you want breakpoints to be different than the default values, we recommend:
- Only setting `sm` and let the system use the default `4/3` ratio.
- If you want a different ratio between breakpoints, setting _all_ breakpoints. 

### Responsive Context
A `ResponsiveContext` is the root of all responsive layouts. It is a container that will report a breakpoint based on its width and children elements will layout things accordingly.

<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Note</h4>
        <p>Responsive layouts will react to parent <code>ResponsiveContexts</code>, not to the app window size. This is a feature. You can have multiple <code>ResponsiveContexts</code> in your app (e.g. multiple resizable windows or viewports) and your elements will react and layout independently of each other.</p>
        <p>If you want your layouts to react to the app size, you can simply create one <code>ResponsiveContext</code> at the top level of your UI hierarchy.</p>
    </div>
</div>

You define your responsive breakpoints through `ResponsiveContext`'s Props.

#### Props
- `int sm`: Min width for Small breakpoint. Extra Small min width is _always_ 0.
- `int md`: Min width for Medium breakpoint.
- `int lg`: Min width for Large breakpoint.
- `int xl`: Min width for Extra Large breakpoint.
- `int xxl`: Min width for Extra Extra Large breakpoint.
- `DOMDescriptor descriptor`: Styling information. Expanded in `Create` method.
- `Children children`: Container's children.
- `Action<float, ResponsiveBreakpoint> onResize`: Callback that gets called when the container gets resized.

### Containers
Containers are the most basic responsive layout element. They are used to contain, pad, and (sometimes) center the content within them.

Getting inspiration from Bootstrap, there are 2 types of Container elements:
- `Container`: It is 100% wide (`width: 100%`) until the specified breakpoint is reached, after which we apply `max-widths` for each of the higher breakpoints. Equivalent to `.container-{breakpoint}` in Bootstrap.
- `FluidContainer`: Full width container (`width: 100%`), spanning the entire width of the viewport. Equivalent to `.fluid-container` in Bootstrap.

If `Container`'s breakpoint is `ExtraSmall` (the default value), it's equivalent to `.container` in Bootstrap and it behaves the same to `Small` breakpoint.

The table below illustrates how each container compares to each other at each breakpoint (assuming default breakpoint values).

<div class="table-responsive my-4">
    <table class="table table-striped">
        <thead>
            <tr>
                <th scope="col"></th>
                <th scope="col">
                    Extra Small
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    Small
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    Medium
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    Large
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    Extra Large
                    <br>
                    <small class="font-monospace opacity-25">width / max-width</small>
                </th>
                <th scope="col">
                    Extra Extra Large
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


#### `FluidContainer` Props



