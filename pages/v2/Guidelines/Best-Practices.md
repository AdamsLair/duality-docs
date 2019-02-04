---
title: "Best Practices"
parent: "/pages/v2/Guidelines"
displayOrder: 100
version: "v2"
---

Ways to keep your code and workflow as clean, error-free and forward-compatible as possible.

# Coding

## Use `Time.TimeMult` where appropriate

Depending on how its configured, Duality might run your game at different frame rates. They may be fixed, or change dynamically. Since your game logic is updated once a frame, but a frame may represent different amounts of passed time, your code will need to account for this.

As a shortcut, Duality provides the static `Time.TimeMult` property, which will tell you how fast the game is currently running. At the expected framerate of 60 FPS, `TimeMult` will be exactly `1.0f`. At 30 FPS, it will be `2.0f`, which essentially means "because the game is running at half speed, each update step needs to account for a time slice of twice the duration".

If anything in your simulation moves, ticks, charges, or is otherwise dependent on the passing of time, you will need to somehow include `Time.TimeMult` into the equation. So, if you call `transform.MoveBy` to move `speed` units to the left each frame, instead let it move `speed * Time.TimeMult` units each frame, and so on - you get the idea.

When properly accounting for variable time steps, your game will be able to run at a constant speed regardless of its current framerate. As a bonus for sticking to this rule, you'll get to control the overall game simulation speed by setting `Time.TimeScale` easily, or pause the game using `Time.Freeze()`

## Getting references to other objects

Say, you're developing the `FooComponent` and need to access a `GameObject` that is not its parent. But how do you get a reference to your target object? Here are some basic options; know their merits and flaws and use them wisely:

### Finding objects by name

An easy, but fragile way that **should be avoided** in almost all cases. 

```csharp 
GameObject obj = this.GameObj.ParentScene.FindGameObject("TheObjectImLookingFor");
obj.DoSomething();
```

* :x: Breaks when the object is renamed.
* :x: Tight coupling between source and target object.
* :x: Not reusable. You will need to change your code on a case-by-case basis.
* :x: Object is searched for in each call, which may or may not affect performance.

### Finding objects using hierarchical relations

Asking for the current object's parent's second child's child. This approach **should be avoided** unless you know exactly what you're doing and why you're doing it. Not recommended in general.

```csharp 
GameObject obj = this.GameObj.Parent.ChildAtIndex(1).ChildAtIndex(0);
obj.DoSomething();
```

* :x: Breaks when the Component is used in a different context than originally intended.
* :x: Tight coupling between the source object and the hierarchy surrounding it.
* :x: Not reusable, unless the same hierarchy is recreated.
* :x: Object graph is queried in each call, which may or may not affect performance.

### Finding objects by Component

Find an object based on the Components that are attached to it. Either use a semantically matching existing Component, or create a new, empty "tag" Component you can attach to the object(s) in question.

```csharp 
GameObject obj = this.GameObj.ParentScene.FindGameObject<BarComponent>();
obj.DoSomething();
```

* :four_leaf_clover: Dynamically retrieves objects based on their traits and behavior.
* :four_leaf_clover: No direct coupling between source and target object.
* :four_leaf_clover: Limited reusability, depending on the context.
* :four_leaf_clover: Works great for groups of Components, or even just Components implementing a certain interface.
* :x: Doesn't really work for a single, specific object.
* :x: Object is searched for in each call, which may or may not affect performance.

### Assigning objects in the editor

Defining a public property in your source Component, which can be assigned by the user in the editor.

```csharp 
public class FooComponent
{
    // Also works fine with Components of any type, or lists of objects.
    private GameObject target;
    public GameObject TargetObject
    {
        get { return this.target; }
        set { this.target= value; }
    }
}
```

* :four_leaf_clover: The reference is already there - no need to search objects at all. Fast!
* :four_leaf_clover: No direct (source code) coupling between source and target object.
* :four_leaf_clover: Really good reusability, the user can assign what is needed and adapt.
* :x: Not a good match in dynamic situations where the target object(s) isn't known until the game runs.

## Use `ContentRef<T>` properly

This is already described [in the article about Resources](../Resource.md), but it's mirrored here because it's so important.

## Deal with null

When retrieving other GameObjects, external or non-required Components, Resources and object instances in general, always assume that they may be `null` and deal with the case gracefully. Letting a null reference Exception occur may be a viable option when a value being `null` is actually considered a fault! However, simply failing to address a viable case of a value genuinely being `null` is something to look out for and fix.

## Mind what you serialize

When you define a custom Component or Resource in Duality, you should ask yourself what kind of information needs to be saved with its Scene or Prefab, or copied with its parent object when cloning it. In many cases, all information is viable for serialization and you don't have to worry about it. However, there are cases where you might want to exclude a field from serialization:

* The field contains **temporary** information that will be recalculated anyway, such as buffers or timer values.
* The field contains a value that is only **established at runtime**, but shouldn't be carried into persistent state. Some event handlers belong to this category, as they expect other objects to resubscribe upon loading and thus won't serialize their subscriber list.
* The field has a Type that **cannot be serialized** (such as `IntPtr`).

In these cases, simply add a `[DontSerialize]` attribute before the field in question.

# Workflow

## Learn to use your Debugger

This might be obvious for some of you, but if you are new to programming and learn it with Duality, this might be the single most important lesson to learn when trying to figure out why something isn't working as expected: Use your debugger. This section can only provide a very limited introduction, but you'll quickly figure out the rest by yourself once you've passed that initial threshold.

* **Starting a Debug session** can be done by either clicking the `Debug Game` button in the Duality editor, or by clicking `Start Debugging` (The "play" button) in your plugin project in Visual Studio.
* **Attaching the Debugger** is a way to debug an application that is already running. This is done automatically when starting a debug session, but if you want to debug your plugin code while it's running in the editor, this is what you need: In Visual Studio, click `Debug / Attach to process...` and select `DualityEditor.exe` to enter debug mode within the Duality editor.
* **Setting a Breakpoint** will allow you to pause the running program at a specific point in your own source code. In Visual Studio, click on the leftmost vertical bar next to your code and a red dot will appear. When in a debug session, this is where the program will now halt.
* **Stepping through code** is what you usually do after hitting a breakpoint. You can step through your code line by line using `Step Over`, or run into a method call using `Step Into`. Wherever you are while you're stepping around, you can hover your cursor over variables to see their value and examine the state of each object.

Of course there's a lot more to is, but this will hopefully get you started.