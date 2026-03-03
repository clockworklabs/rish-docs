---
title: Windows
slug: windows
sections:
    - Context
    - Window
    - Dragging
icon: window-restore
---

Because UI Toolkit lacks a `z-index` property and the visual order is determined by the order in the hierarchy, we have to get creative when we want elements to be rendered on top of everything. Windows are one such situation.

## Context
A `WindowsContext` ancestor is necessary for `Window`s to work. It handles windows positioning and order.

### Props
- `bool forceFit`: If true, windows will be forced to fully stay inside the content rect of the `WindowContext`.
- `int safeZoneSize`: The safe zone size in pixels. If `forceFit` is false, windows will be forced to never leave the safe zone.
- `bool hideAllWindows`: If true, all windows are hidden (even if they're open).
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `Children children`: Children.

## `Window`
The `Window` element actually returns nothing in its `Render` method: `protected override Element Render() => Element.Null;`. Instead it communicates with a `WindowsContext` ancestor and the context will add `content` as one of its children.

<div class="alert alert-info" role="alert">
    In a future release, we will add support to resize windows.
</div>

<div class="alert alert-success" role="alert">
    Rootstrap provides a <code>SimpleWindow</code> wrapper.
</div>

### Props
- `ulong? guid`: If provided, it must be unique and it allows `WindowsContext` to identify the `Window` even after hierarchy changes. It is recommended to always set it.
- `bool open`: Whether or not the window is open.
- `bool draggable`: Whether or not the window can be dragged.
- `Element content`: The content that `WindowsContext` will add when the window is open.
- `bool alwaysOnTop`: Whether or not the window should always be on top of all the others.
- `Vector2? offset`: Offset relative to the top left corner of the `WindowsContext` content rect. If set, the default dragging system will be skipped and this offset will always be used.
- `Action<WindowDragData> onDrag`: Callback that gets called when the window is dragged.

## Dragging
A `Window` comunicates and passes _upwards_ a `content` to a `WindowsContext`. It's not listening to input events, it's not rendering anything, it's just a logical "ghost" element. 

To allow `Window`s to be dragged, you need to set `draggable` to `true` and `content` must have a `WindowHeader` child. The `WindowHeader` will listen to input events and communicate with the `WindowsContext`.

<div class="callout-block callout-block-info">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Custom Logic</h4>
        <p>In advanced scenarios where you need custom behavior, you can set `offset` in a `Window` and implement your own dragging or positioning system.</p>
    </div>
</div>