---
layout: docs
title: Installation
sections:
  - Dependencies
  - Set Up
order: 1
---

Installing Rish is very simple. You just need to add (via the Package Manager or by modifying `manifest.json` under the `Packages` folder) the package `https://github.com/clockworklabs/rish#[targe-version]`.

## Dependencies
Rish only depends on Unity 2022.3+ and `com.unity.collections#1.2.4`+.

## Set Up
You can have more than one Rish app running at the same time. A common example of this would be an app for all the regular UI in screen space and a separate UI for all world space HUDs.

To set up a Rish app, you need to add the component `RishRoot` to any `GameObject` (it's usually best practice to add it to a GameObject which sole purpose is to serve as a starting point for Rish). This component needs a `UIDocument` and will automatically add one to the GameObject if missing. You'll need to assign a Panel Settings to the UIDocument and a Root App to RishRoot. The root app must be a class that implements the `IApp` interface.

{% highlight csharp %}
public class App : IApp
{
    Element IApp.GetRoot(bool recovered) => H1.Create(text: "Hello, world!");
}
{% endhighlight %}

A Rish app isn't an element, it's just a starting point and it loads the initial element for the whole app. A very common setup is to have a Root element with state (and maybe properties).

{% highlight csharp %}
public partial class App : IApp
{
    Element IApp.GetRoot(bool recovered) => Root.Create(text: "Hello, world!");

    private partial class Root : RishElement<RootProps, RootState>, IMountingListener
    {
        public Root
        {
            if(StaticData.IsLoaded)
            {
                OnLoadingProgress(1);
            } else {
                StaticData.OnLoadingProgress += OnLoadingProgress;
            }
        }

        protected override Element Render() {
            if(State.loaded) {
                return GameScreen.Create();
            }

            return LoadingScreen.Create(progress: State.loadingProgress);
        }

        private void OnLoadingProgress(float v)
        {
            var state = State;
            state.loadingProgress = v;
            State = state;
        }
    }
    [RishValueType]
    public struct RootProps {
        
    }
    [RishValueType]
    public struct RootState {
        public float loadingProgress;

        public bool loaded => loadingProgress >= 1;
    }
}
{% endhighlight %}