---
layout: docs
title: Installation
sections:
  - Dependencies
  - Set Up
  - Samples
order: 1
icon: download
---

Installing Rish is very simple. You just need to add (via the Package Manager or by modifying `manifest.json` under the `Packages` folder) the package `https://github.com/clockworklabs/rish#[target-version]`.

## Dependencies
Rish depends on Unity 2022.3+, `com.unity.collections#1.2.4`+ and `io.clockworklabs.sappy#1.0.0`+.

### Rishenerator (Roslyn Source Generator)
We do not recommend using Rish without Rishenerator, the included Roslyn Source Generator. To add it to the project, search for Rish in the Package Manager, go to Samples and import Rishenerator. In the inspector, make sure the imported `Rishenerator.dll` has the `RoslynGenerator` label and has no platforms selected.

While strickly not necessary, because you can write all the necessary code yourself, we seriously can't stress enough how much better is to use Rishenerator: it reduces drastically the development time, makes the code much easier to mantain and much less error prone, it improves the readability of the code and, most importantly, it automatically handles all the necessary stuff to keep Rish fast and memory efficient. For the rest of the guide, we'll assume Rishenerator is being used. 

## Set Up
You can have more than one Rish app running at the same time. A common example of this would be an app for all the regular UI in screen space and a separate UI for all world space HUDs.

Each app will create a separate UI Toolkit tree.

To set up a Rish app, you need to add the component `RishRoot` to any `GameObject` (it's usually best practice to add it to a GameObject which sole purpose is to serve as a starting point for Rish). This component needs a `UIDocument` and will automatically add one to the GameObject if missing. You'll need to assign a Panel Settings to the UIDocument and a Root App to RishRoot. The root app must be a class that implements the `IApp` interface.

{% highlight csharp %}
public class App : IApp
{
    Element IApp.GetRoot(bool recovered) => H1.Create(text: "Hello, world!");
}
{% endhighlight %}

A Rish app isn't an element, it's just the starting point and it's in charge of defining the initial element for the whole app. A very common setup is to have a Root element with state (and maybe properties).

{% highlight csharp %}
public partial class App : IApp
{
    Element IApp.GetRoot(bool recovered) => Root.Create();

    private partial class Root : RishElement<NoProps, RootState>, IMountingListener
    {
        void IMountingListener.ComponentDidMount() {
            if(StaticData.IsLoaded)
            {
                OnLoadingProgress(1);
            } else {
                StaticData.OnLoadingProgress += SetLoadingProgress;
            }
        }
        void IMountingListener.ComponentWillUnmount() { }ß

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

## Samples
We recommend installing Roots, importing the samples, play around and look at the code for a little bit before moving on with the next sections.