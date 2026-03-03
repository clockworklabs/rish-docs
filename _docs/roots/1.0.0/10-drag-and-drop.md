---
title: Drag and Drop
slug: drag-and-drop
sections:
    - Context
    - Draggable
    - DropArea
icon: hand-back-fist
---

Because UI Toolkit lacks a `z-index` property and the visual order is determined by the order in the hierarchy, we have to get creative when we want elements to be rendered on top of everything. Drag & Drop is one such situation.

## Context
A `DragAndDropContext` ancestor is necessary for `Draggables`s to work. When an element is being dragged, it will add it at the end of its hierarchy (on top of everything else) and will handle its positioning.

### Props
- `Action<bool> onDrag`: Callback that gets called when dragging begins or ends.
- `VisualAttributes descriptor`: Styling information. Expanded in `Create` method.
- `Children children`: Children.

## `Draggable`
The `Draggable` element is kind of special: it's a generic RishElement. The type argument determines the type of information the draggable element holds and it must be a `struct` type.

### Props
- `Element content`: The `Element` that is used by default.
- `Element contentWhileDragging`: The `Element` that is used while it's being dragged. Useful to show a grayed out variation or something similar. If invalid, it will fallback to `content`.
- `Element secondaryContentWhileDragging`: The `Element` that is used while it's being dragged with the right mouse button. Useful to show a grayed out variation or something similar. If invalid, it will fallback to `contentWhileDragging`.
- `Element draggedElement`: The `Element` that follows the cursor.
- `Element draggedAcceptedElement`: The `Element` that follows the cursor when hovering a valid `DragArea`. If invalid, it will fallback to `draggedElement`.
- `Element draggedRejectedElement`: The `Element` that follows the cursor when hovering an invalid `DragArea`. If invalid, it will fallback to `draggedElement`.
- `Element secondaryDraggedElement`: The `Element` that follows the cursor when dragging with the right mouse button.
- `Element secondaryDraggedAcceptedElement`: The `Element` that follows the cursor when hovering a valid `DragArea` when dragging with the right mouse button. If invalid, it will fallback to `secondaryDraggedElement`.
- `Element secondaryDraggedRejectedElement`: The `Element` that follows the cursor when hovering an invalid `DragArea` when dragging with the right mouse button. If invalid, it will fallback to `secondaryDraggedElement`.
- `T info`: The information the element holds and drops.
- `T? secondaryInfo`: The information the element holds and drops when dragging with the right mouse button. If null, it will fallback to `info`.
- `Action onDragAction`: Callback that gets called when the drag action begins.
- `Action onDropAction`: Callback that gets called when the element is dropped.
- `Action dropNowhereAction`: Callback that gets called when the element is dropped over empty space.

## `DropArea`
`Draggable` elements must be dropped over compatible `DragArea`s. The `DragArea` element is also generic and their type argument must match for them to be compatible with each other.

Not only `Draggable`s and `DropArea`s must be compatible, but the `DropArea` also has to validate if the information being dragged is accepted or not. You can imagine a scenario where an item is being dragged but it can only be dropped in certain inventories or slots; some `DropArea`s will accept the item and some will reject it, even though the types match. 

### Props
- `Element content`: The `Element` that is used by default.
- `Element highlightedContent`: The `Element` that is used when a compatible and acceptable `Draggable` is being dragged. Useful to show a drop call to action. If invalid, it will fallback to `content`.
- `Element hoveredAcceptedContent`: The `Element` that is used when a compatible and acceptable `Draggable` is hovering the `DropArea`. If invalid, it will fallback to `highlightedContent`.
- `Element hoveredRejectedContent`: The `Element` that is used when a compatible but rejected `Draggable` is hovering the `DropArea`. If invalid, it will fallback to `content`.
- `Predicate<T> canDropAction`: The predicate that is called to determine if the `DropArea` accepts or rejects the `Draggable` information.
- `Action<T> dropAction`: The callback that is called when an accepted `Draggable` is dropped onto the `DropArea`.
- `Action<T> hoverStarted`: The callback that is called when a compatible `Draggable` starts hovering the `DropArea`. Useful to play a sound effect or some other kind of feedback.
- `Action hoverEnded`: The callback that is called when a compatible `Draggable` stops hovering the `DropArea`. Useful to play a sound effect or some other kind of feedback.