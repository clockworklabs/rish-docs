---
title: Rootstrap
slug: rootstrap
sections:
    - Elements
    - Utilities
icon: bootstrap
icon-style: brands
---

You can import Rootstrap from the Samples tab in the Package Manager.

1. Open the **Package Manager**.
2. Select the **Roots** package.
3. Go to the **Samples** tab and import **Rootstrap**.

## Elements
It contains many UI Elements that you can use as a reference or as a starting point. We used Bootstrap (hence the name) as a reference.
- **`Alert`:** A simple alert container with various styles and colors.
- **`Button`:** A simple button with various styles and colors.
- **`Card`:** A simple container with a header and a body.
- **`H1`:** A label intended to be used as a heading level 1. It returns a Label with the `h1` class name.
- **`H2`:** A label intended to be used as a heading level 2. It returns a Label with the `h2` class name.
- **`H3`:** A label intended to be used as a heading level 3. It returns a Label with the `h3` class name.
- **`H4`:** A label intended to be used as a heading level 4. It returns a Label with the `h4` class name.
- **`H5`:** A label intended to be used as a heading level 5. It returns a Label with the `h5` class name.
- **`H6`:** A label intended to be used as a heading level 6. It returns a Label with the `h6` class name.
- **`Body`:** A label intended to be used as body. It returns a Label with the `body` class name.
- **`Small`:** A label intended to be used as small body. It returns a Label with the `small` class name.
- **`Rule`:** A small line divider. Horizontal in column layouts and vertical in row layouts.
- **`Material Symbols`:** ...
- **`ProgressBar`:** A simple progress bar with various styles and colors.
- **`SimpleScrollView`:** A minimalist scroll view with a scroll bar. It has `VScrollView` and `HScrollView` wrappers for vertical and horizontal scroll views respectively.
- **`SimpleTooltip`:** An element that can show a simple text tooltip.
- **`SimpleWindow`:** A window with a header in the style of a `Card`.
- **`Spinner`:** A simple spinner to signify a loading progress. It has two styles.

## Utilities
Rootstrap comes with a lot of class names that basically mimics Bootstrap's utilities 1-to-1. In particular:
- [Background](https://getbootstrap.com/docs/5.3/utilities/background/).
- [Borders](https://getbootstrap.com/docs/5.3/utilities/borders/) (except some USS limitations).
- [Colors](https://getbootstrap.com/docs/5.3/utilities/colors/).
- [Display](https://getbootstrap.com/docs/5.3/utilities/display/) (although just `flex` or `none` due to USS limitations).
- [Opacity](https://getbootstrap.com/docs/5.3/utilities/opacity).
- [Position](https://getbootstrap.com/docs/5.3/utilities/position) (only `absolute` or `relative` due to USS limitations).
- [Sizing](https://getbootstrap.com/docs/5.3/utilities/sizing/).
- [Spacing](https://getbootstrap.com/docs/5.3/utilities/spacing/).
- [Text](https://getbootstrap.com/docs/5.3/utilities/text/).
- [Visibility](https://getbootstrap.com/docs/5.3/utilities/visibility/).

There's also a static `Utilities` class to create and use these class names in a more robust, less error-prone way.
