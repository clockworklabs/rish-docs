---
title: Forms
slug: forms
sections:
    - Form
    - InputField
    - Submit Button
icon: pen-to-square
---

## `Form`
The `Form` element is just a logical container. Children elements can use it to get a keyboard focus index (for `Tab` navigation) based on their position in the hierarchy. It also listens to `Return` key down events to submit.

### Props
- `Element content`: The content.
- `Action submitAction`: The callback that gets called when the form is submitted.

## Input Field
The `InputField` wraps around UI Toolkit's `InputField`. Many decisions by the UI Toolkit team makes it complicated to implement a fully custom input field independent from their implementation.

### Props
- `InputField.Type type`: `Text`, `Integer`, `Long` or `Float`.
- `VisualAttributes visualAttributes`: Styling information. Expanded in `Create` method.
- `VisualAttributes textInputVisualAttributes`: Styling information for the `TextInput` child. Expanded in `Create` method.
- `VisualAttributes textElementVisualAttributes`: Styling information for the `TextElement` child. Expanded in `Create` method.
- `RishString value`: Input field value.
- `bool updateOnEveryKeystroke`: If true, `onChange` is called on every keystroke. If false, `onChange` is called only when element loses keyboard focus or Return key is pressed.
- `bool multiline`: Whether or not this InputField is multiline or not.
- `bool isPassword`: If true, the value will be visibly offuscated (***).
- `bool readOnly`: If true, the value can't be modified.
- `int? maxLength`: The max length.
- `bool? multiClickInteraction`: If true, double click will select words and triple click will select lines.
- `bool? selectOnFocus`: If true, all content will be selected when the element receives focus.
- `bool autoFocus`: If true, the element will receive keyboard focus when mounted.
- `bool richTextEnabled`: Whether or not rich text tags get parsed or not.
- `bool parseEscapeSequences`: Wheter or not escape sequences get parsed or not.
- `Color? cursorColor`: The color of the cursor.
- `Color? selectionColor`: The color of the selection box (which is always rendered on top of the text).
- `Action<string> onChange`: The callback that gets called when the value changes.

## Submit Button
The `AbstractButton` has a `submitForm` property. When set to true, clicking on that button will call the enclosing `Form` `submitAction` callback.