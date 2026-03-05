---
title: Dropdowns
slug: dropdowns
sections:
    - Context
    - DropdownButton
icon: square-caret-down
---

Because UI Toolkit lacks a `z-index` property and the visual order is determined by the order in the hierarchy, we have to get creative when we want elements to be rendered on top of everything. Dropdowns are one such situation.

## Context
A `DropdownContext` ancestor is necessary for `Dropdown`s to work. When a dropdown menu is needed, it will add it at the end of its hierarchy (on top of everything else) and will handle its positioning.

It will only show one dropdown at a time.

### Props
- `bool forceFit`: If true, tooltips are forced to fit within the `DropdownContext` content rect.
- `Action<bool> onShow`: Callback that gets called when a dropdown menu is shown or hidden.
- `VisualAttributes visualAttributes`: Styling information. Expanded in `Create` method.
- `Children children`: Children.

## `DropdownButton`
The `DropdownButton` element wraps around `AbstractButton` and it adds a new `open` style. It listens to pointer click events and communicates with a `DropdownContext` ancestor to show the `menu` in the right place.

<div class="alert alert-success" role="alert">
    Rootstrap provides a <code>SimpleDropdown</code> wrapper.
</div>

### Props
- `bool interactable`: Whether the button is enabled or not.
- `Element normal`: The `Element` that is used by default.
- `Element hovered`: The `Element` that is used when the button is being hovered. If invalid, it will fallback to `normal`.
- `Element pressed`: The `Element` that is used when the buttons is being pressed. If invalid, it will fallback to `hovered`.
- `Element disabled`: The `Element` that is used when the button is disabled. If invalid, it will fallback to `normal`.
- `Element open`: The `Element` that is used when the menu is open. If invalid, it will fallback to the right pointer state (`normal`, `hovered` or `pressed`).
- `Element menu`: The dropdown menu element to show.