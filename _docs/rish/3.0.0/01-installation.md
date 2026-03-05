---
title: Installation
slug: installation
sections:
  - Setup
  - Next Steps
icon: download
---

Installing Rish is simple. You can add the package via the Unity Package Manager using the Git URL, or by modifying your `manifest.json` file directly.

Add the following package URL: `https://github.com/clockworklabs/rish#[target-version]`.

#### Dependencies
Rish requires the following dependencies to function correctly:
- Unity: 2022.3 or higher.
- Collections: `com.unity.collections` (version `1.2.4+`).
- [Sappy](https://github.com/clockworklabs/SappyEvents): `io.clockworklabs.sappy` (version `1.0.0+`).

### Rishenerator (Source Generator)
Rish comes with a Roslyn Source Generator called **Rishenerator**. While it is technically possible to write all your Rish code manually, **we strongly recommend using Rishenerator**.

<div class="callout-block callout-block-success">
    <div class="content">
        <h4 class="callout-title"><i class="fa-solid fa-circle-info"></i>Why use Rishenerator?</h4>
        <ul>
            <li><strong>Drastically reduces development time</strong> by automating boilerplate.</li>
            <li><strong>Eliminates errors</strong> associated with manual implementation.</li>
            <li><strong>Optimizes performance</strong> and memory efficiency automatically.</li>
        </ul>
    </div>
</div>

#### Installing Rishenerator
1. Open the **Package Manager**.
2. Select the **Rish** package.
3. Go to the **Samples** tab and import **Rishenerator**.
4. In your project assets, locate the imported `Rishenerator.dll`.
5. **Crucial Step:** In the Inspector for the DLL:
  - Ensure the **Select platforms for plugin** list is empty (no platforms selected).
  - Add (if not present already) the `RoslynAnalyzer` label.

<div class="alert alert-danger" role="alert">
    <i class="fa-solid fa-triangle-exclamation"></i> The rest of this guide assumes Rishenerator is enabled!
</div>

## Setup
You can run multiple Rish apps simultaneously (e.g., one for Screen Space UI and another for World Space HUDs). Each app manages its own UI Toolkit tree.

To set up a Rish App in your scene:
1. Create an empty **GameObject** (e.g., named "UI_Root").
2. Add the `RishRoot` component to it. This will automatically add a `UIDocument` component if one is missing.
3. Assign your **Panel Settings** to the `UIDocument`.
4. Create a class that implements `IApp` and assign it to the `RishRoot`.
5. Assign all the USS style sheets your app will need to the `RishRoot`. 

### Defining an App
A Rish App is the entry point for your UI. It is not an Element itself; rather, it defines the root element for the entire tree.

#### Basic Example
For a stateless, simple UI:

{% highlight csharp %}
public class App : IApp
{
    // The 'recovered' parameter indicates if the app has been recovered and relaunched after a crash.
    // You can use this to show a message or something.
    Element IApp.GetRoot(bool recovered) => H1.Create(text: "Hello, world!");
}
{% endhighlight %}

#### Advanced Example (Stateful Root)
A common pattern is to have a "Root" element that manages high-level state (like loading progress) and mounts the rest of the application.

{% highlight csharp %}
public partial class App : IApp
{
    Element IApp.GetRoot(bool recovered) => Root.Create();

    // The internal Root element manages the application state
    private partial class Root : RishElement<NoProps, RootState>, IMountingListener
    {
        void IMountingListener.ElementDidMount() {
            if(StaticData.IsLoaded)
            {
                // If data is already ready, set progress immediately
                SetLoadingProgress(1);
            } else {
                // Subscribe to loading progress events
                StaticData.OnLoadingProgress += SetLoadingProgress;
            }
        }
        void IMountingListener.ElementWillUnmount() {
            // Unsubscribe from loading progress events
            StaticData.OnLoadingProgress -= SetLoadingProgress;
        }

        protected override Element Render() {
            if(State.loaded) {
                return GameScreen.Create();
            }

            return LoadingScreen.Create(progress: State.loadingProgress);
        }
    }

    [RishValueType]
    public struct RootState {
        public float loadingProgress;

        public bool loaded => loadingProgress >= 1;
    }
}
{% endhighlight %}

## Next Steps
Now that you have the library installed, we recommend installing [**Roots**](/docs/roots/quick-start) and importing its samples to see working code in action.