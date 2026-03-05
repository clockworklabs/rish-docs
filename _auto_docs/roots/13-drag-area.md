---
title: Drag Area
slug: drag-area
sections:
    - Props
icon: up-down-left-right
---

The `DragArea` allows its `content` to be dragged around inside of it using the mouse wheel or pointer interactions.

The `content` is added to a Holder element that can be independently styled from the `DragArea` itself.

## Props
- `VisualAttributes visualAttributes`: Styling information. Expanded in `Create` method.
- `VisualAttributes holderVisualAttributes`: Styling information for the holder element. Expanded in `Create` method.
- `DragArea.VisualBehavior behavior`: `Default` (content is allowed to go a little over the limit and returns to the limit when released), `Stretchy` (content stretches when trying to go over the limit), `Clamped` (content is not allowed to go over the limits).
- `DragArea.AxisOfFreedom axisOfFreedom`: `None`, `Vertical`, `Horizontal` or `VerticalAndHorizontal`.
- `Translate initialOffset`: Initial offset of `content` within the `DragArea`.
- `Element content`: The visual content of the `DragArea`.
- `Length? extraMargin`: TODO
- `float? mouseWheelMultiplier`: Multiplier to adjust mouse wheel sensitivity. If not set, 30 is used by default.
- `Vector2? offset`: The offset of `content` within the `DragArea`. If not set, `content` position is fully controlled by `DragArea`.
- `Action<Vector2> onOffset`: Callback that gets called when `content` is dragged around. 