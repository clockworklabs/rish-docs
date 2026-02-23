---
title: Buttons
slug: buttons
sections:
  - Props
  - Rootstrap
icon: hand-pointer
---

The `AbstractButton` element is a foundational button RishElement. It supports up to 4 visual styles: normal, hovered, pressed and disabled. It can easily be set up as a submit button within a Form. It can receive keyboard focus.

## Props
- `bool interactable`: Whether the button is enabled or not.
- `Action action`: The callback that is called when pressed (unless `submitsForm` is `true`).
- `Element normal`: The `Element` that is used by default.
- `Element hovered`: The `Element` that is used when the button is being hovered. If invalid, it will fallback to `normal`.
- `Element pressed`: The `Element` that is used when the buttons is being pressed. If invalid, it will fallback to `hovered`.
- `Element disabled`: The `Element` that is used when the button is disabled. If invalid, it will fallback to `normal`.
- `bool focusable`: Whether the button can receive keyboard focus or not.
- `bool autoFocus`: Whether the button should receive keyboard focus when mounted. If `focusable` is false, it is ignored.
- `bool submitsForm`: When true, if the button is a part of a Form, it will call the submit action of the Form.

## Rootstrap
Rootstrap provides a `SimpleButton` wrapper.