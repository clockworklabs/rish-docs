---
title: Installation
slug: installation
sections:
  - Dependencies
  - Set Up
  - Samples
icon: download
---

Installing Roots is simple. You can add the package via the Unity Package Manager using the Git URL, or by modifying your `manifest.json` file directly.

Add the following package URL: `https://github.com/clockworklabs/roots#[target-version]`.

#### Dependencies
Roots requires the following dependencies to function correctly:
- [Rish](/docs/rish/quick-start): `io.clockworklabs.rish` (version `3.0.0+`).
- [Motion](https://github.com/clockworklabs/motion): `io.clockworklabs.motion` (version `1.7.9+`).

## Setup
Roots requires minimal setup for all of its moving parts to work:
1. Add `RootsSetup` component to the GameObject that contains your App's `RishRoot`.
  - This component is only needed to use the more advanced `ResponsiveStyleSheets`.
2. Add an `AssetsLoader` component to bridge between your app's assets pipeline and your UI app.
  - `AssetsLoader` is an abstract class. Roots provides a `ResourcesLoader` implementation that loads assets from Resources. You can implement your own loader that works for your app.
3. Animated elements use [Motion](https://github.com/clockworklabs/motion). For your motions to be stepped, add a `MotionAutoUpdate` component (in more advanced scenarios, you may want to manually call `DoMotion.Step`).

## Samples
Roots comes with samples showing a wide range of UI Elements (from simple buttons to complex scroll views or responsive layouts).

1. Open the **Package Manager**.
2. Select the **Roots** package.
3. Go to the **Samples** tab and import **Rootstrap** and **Samples**.
4. Open the newly imported `Samples` scene and enter Play Mode.

The Samples scene supports changing the dimensions of the samples container to interact with responsive elements. It also has a button to see the relevant code for each sample.

We encourage you to play around.