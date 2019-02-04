---
title: "Components"
category: "basics"
displayOrder: 0
version: "v3"
---

A **Component** is how game logic is defined in Duality. But what exactly is a Component?

# Building Blocks
Components are the basic building blocks of your game: Each of them has a certain purpose and they interact with each other in order to produce complex behavior. One Component might provide a position in space, another might use that to draw a sprite. You can define your own Components to introduce your own code and logic into the engine.

A Component can be attached to a GameObject, it can run code, you can get query the active Scene for instances of it, and so on. It is not possible to add more than one Component of the same type to a GameObject, in order to avoid ambiguity, and a single Component may not belong to multiple GameObjects. Components can also require another Component of a specified type to be present in its parent GameObject, which will be enforced by the editor.

To make a custom component, you must derive (inherit) from it:
```csharp
using Duality;

namespace MyNamespace
{
	public class MyCustomComponent : Component
	{
	}
}
```
There! Now we have a shiny new custom component, ready for action!

## Sidenote: The Component Base Class
Every Component has a few properties and functions that you might find handy when writing a game in Duality:

**Properties:**
* `bool Active`: Whether the component and all of its ancestors / parent objects are currently active in the scene.
* `bool Disposed`: Whether the component has been "deleted" or removed. It's not something you usually have to be aware of, but when you store a reference to some component, this is how you know whether it's still around. If this is false, you can treat it the same as a `null` reference.
* `GameObject GameObj`: The GameObject this Component belongs to; its parent.
* `Scene Scene`: The Scene in which this Component operates.

**Methods:**
* `void DisposeLater()`: Schedule the Component for disposal (deletion/removal) at the end of the current update cycle. This function will only dispose of the Component itself, not its parent GameObject or any other sibling Components. To dispose the entire GameObject, call its own `DisposeLater` method.

This is not an exhaustive list, but a quick heads-up of some of the functionality that you might find interesting.

# Ready for Action
...and how do we get Components to "act", exactly?

Components in Duality do not have any basic initialization, shutdown, or update functions associated with them. Instead of overriding base class methods or using hardcoded method names for certain purposes, Duality classifies and gives functionality to components based on which interfaces they implement. By default, it will consider a Component to be passive. If you want it to do something each frame, or react to certain events, you would implement an interface that represents this.

There are two basic interfaces you should be aware of: `ICmpInitializable` and `ICmpUpdatable`.

# ICmpInitializable
As the name suggests, ICmpInitializable is the interface to implement if you want your custom component to initialize, and / or shutdown. The `OnActivate` method will be invoked whenever the Component switches from a previously inactive state to an active one, which can happen in any of the following ways:

- The Scene to which the Component belongs is activated / entered.
- The GameObject to which the Component belongs is added to an active Scene.
- The Component is added to a GameObject in an active Scene.
- The Component itself is set from `Active` `false` to `true`.
- The GameObject to which the Component belongs, or any of its parent GameObjects is set from `Active` `false` to `true`.

And of course, `OnDeactivate` will be invoked in each of the opposite cases. To simplify matters, you generally do not need to care about which specific event triggered either of the two methods, since all your code generally cares about is when its activation state changes. Duality guarantees that Components are start and end their existence in a deactivated state, and that each `OnActivate` call is followed by an eventual `OnDeactivate` call.

Let's extend our earlier example:
```csharp
using Duality;
using Duality.Resources;

namespace MyNamespace
{
    public class MyCustomComponent : Component, ICmpInitializable
    {
        void ICmpInitializable.OnActivate()
        {
            DualityApp.Sound.PlaySound(Sound.Beep);
            Logs.Game.Write("...and so it begins.");
        }

        void ICmpInitializable.OnDeactivate()
        {
            DualityApp.Sound.PlaySound(Sound.Beep);
            Logs.Game.Write("...and so it ends.");
        }
    }
}
```
So, what did we just change? If you add this component to an object and run the game in the editor sandbox, you will hear a beep sound when pressing the play button, and another one when hitting stop - each of them accompanied by a console log. The activate and deactivate handlers above wrap the active lifetime of its component. In general, anything implementing ICmpInitializable interface must implement two functions, `OnActivate()` and `OnDeactivate()`. 

With that out of the way, the earlier example hopefully makes a little bit more sense. On initialization, we play a sound and write a message to the log. On shutdown, we do the same.

# ICmpUpdatable
This is probably the interface you will be seeing the most of when working with Duality. Any Component implementing this interface will be able to run its own per-frame logic. It's also the easiest one to implement; no parameters to check, no bureaucracy. Just a function and its code:
```csharp
using Duality;

namespace MyNamespace
{
    public class MyCustomComponent : Component, ICmpUpdatable
    {
        private int frameCounter = 0;

        void ICmpUpdatable.OnUpdate()
        {
            // Example: We'll count how many frames have passed.
            this.frameCounter++;
        }
    }
}
```
That's the gist of `ICmpUpdatable`, even though you probably want to do something else than counting frames in a real-world use case. The `OnUpdate` method is where you can put everything that you want to be executed each frame.

# RequiredComponent
Just as formerly mentioned, a Component can require that another Component of a specified type has to be present in the same GameObject. The Duality editor will then enforce this requirement by adding all required Components when adding this Component. Think of a sprite that can only render itself if there is also a `Transform` providing a location in space - or an actor that can only animate if there is a `SpriteRenderer` that displays the current frame.

You can accomplish the above by adding the `RequiredComponent` attribute to your Component class:
```csharp
using Duality;
using Duality.Components.Renderers;

namespace MyNamespace
{
    [RequiredComponent(typeof(SpriteRenderer))]
    public class MyCustomComponent : Component, ICmpInitializable
    {
        private SpriteRenderer spriteRenderer;

        void ICmpInitializable.OnActivate()
        {
            this.spriteRenderer = this.GameObj.GetComponent<SpriteRenderer>();
        }

        void ICmpInitializable.OnDeactivate()
        {
            // It's not really required to nullify references.
            // But this *is* where you clean up, so might as well
            // do some cleaning up.
            this.spriteRenderer = null;
        }
    }
}
```

In this example, we get a reference to the `SpriteRenderer` attached to the parent GameObject on activation and assign it to `spriteRenderer`. Quite a common occurrence when writing code for Duality games. We nullify the reference on deactivation.

An important note: `RequiredComponent` is only an *editor-enforced* feature. It exists to make it easier to set up different kinds of GameObjects within the editor. It's not required to be able to use the `GetComponent<T>()` function.

# Summary
A Component is the basic building block of logic in your Duality-powered game, and it's quite easy to add functionality to it. How you structure it is up to you - nothing prevents you from putting all the logic into a single Component and there are situations where this can actually be useful. In other cases, you may want to move some common functionality to its own Component and re-use it in various places.

Even though you are by no means locked into it, "composition over inheritance" is a principle that's at the core of Duality's Component system, and it can be used to great effect to power GameObjects in your game. For example, make any GameObject able to take damage with a `Destructible` component, or make it a damage dealer with a `DamageDealer` component. The only limit is your imagination (and, well, CPU clock cycles and memory).