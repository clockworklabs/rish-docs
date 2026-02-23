---
title: Images
slug: images
sections:
  - Props
icon: image
---

The `Image` element is a foundational image VisualElement. It inherits from `UnityEngine.UIElements.Image`.

## Props
- `Texture2D texture`: `Texture2D` to show.
- `Sprite sprite`: `Sprite` to show. Less precedence than `texture`.
- `VectorImage vector`: `VectorImage` to show. Less precedence than `sprite`.
- `RenderTexture renderTexture`: `RenderTexture` to show. Less precedence than `vector`.
- `bool useBackground`: Whether the image should use USS background properties or not. 9-slice Sprites will automatically use the background properties.
- `RishString textureAddress`: Address to load a `Texture2D` (via `AssetsLoader`). Can be set in USS Style Sheet with `--props-texture`.
- `RishString spriteAddress`: Address to load a `Sprite` (via `AssetsLoader`). Can be set in USS Style Sheet with `--props-sprite`. Less precedence than `textureAddress`.
- `RishString vectorAddress`: Address to load a `VectorImage` (via `AssetsLoader`). Can be set in USS Style Sheet with `--props-vector`. Less precedence than `spriteAddress`.
- `RishString renderTextureAddress`: Address to load a `RenderTexture` (via `AssetsLoader`). Can be set in USS Style Sheet with `--props-render-texture`. Less precedence than `vectorAddress`.
- `ScaleMode scaleMode`: `ScaleMode` to use when rendering. Can be set in USS Style Sheet with `--props-scale-mode`.
- `Color tintColor`: `Color` to multiply with. Can be set in USS Style Sheet with `--props-tint-color`.
- `ImageSize width`: Width. Can be set in USS Style Sheet with `--props-image-width`. If set, it has higher precedence than similar USS properties (like `width` or `min-width`).
- `ImageSize height`: Height. Can be set in USS Style Sheet with `--props-image-height`. If set, it has higher precedence than similar USS properties (like `height` or `min-height`).