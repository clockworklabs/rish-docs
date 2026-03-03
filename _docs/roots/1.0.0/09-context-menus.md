---
title: Context Menus
slug: context-menus
sections:
    - Context
    - Contextual
icon: bars
---

Because UI Toolkit lacks a `z-index` property and the visual order is determined by the order in the hierarchy, we have to get creative when we want elements to be rendered on top of everything. Context Menus are one such situation.

## Context
A `ContextualContext` ancestor is necessary for `Contextual`s to work. When a context menu is needed, it will add it at the end of its hierarchy (on top of everything else) and will handle its positioning.

It will only show one context menu at a time.

### Props
- `Action<bool> onShow`: Callback that gets called when a contextual menu is shown or hidden.
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `Children children`: Children.

## `Contextual`
The `Contextual` element responds to pointer events and communicates with a `ContextualContext` ancestor to show the `menu` in the right place.

<div class="alert alert-success" role="alert">
    Rootstrap provides a <code>SimpleContextual</code> wrapper.
</div>

### Props
- `Element normal`: The `Element` that is used by default.
- `Element hovered`: The `Element` that is used when the button is being hovered. If invalid, it will fallback to `normal`.
- `Element pressed`: The `Element` that is used when the buttons is being pressed. If invalid, it will fallback to `hovered`.
- `Element menu`: The dropdown menu element to show.
- `ContextualAnchor menuAnchor`: Anchor to position the context menu.
- `bool menuAnchorCanChange`: If true, a context menu will change anchor to fit the menu within the `DropdownContext` content rect. If false, an offset will be applied.