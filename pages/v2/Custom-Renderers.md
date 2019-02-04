---
title: "Custom Renderers"
category: "advanced"
displayOrder: 0
version: "v2"
---

This article will introduce you to writing custom renderer Components.

# What is a "Renderer Component"?

When you run your game, the things you see on screen are a visual representation of the objects you defined and created both in the editor and code. While Duality provides the facilities to talk to the render pipeline and ultimately draw things on the screen, it is the job of a renderer Component to _use_ those facilities and actually draw something. 

Depending on how you implement a renderer Component, it can display just about anything: A sprite, geometric shapes, a particle system, a whole tilemap, and so on. It may render in world space or screen space and it is not restricted to the concept of a single GameObject, or object at all.

# Defining a Renderer Component

The bottom line of creating a custom renderer is implementing the `ICmpRenderer` interface. Alternatively, you can derive from `Renderer`, which is an abstract Component type that implements some of the boilerplate code you need for your usual world-space object rendering.

In both cases, you will have to talk to the `IDrawDevice` interface - it's the API that Duality provides to let you draw things.

## Option A: Implementing `ICmpRenderer`

Choosing the bottom line approach, implementing the `ICmpRenderer` interface means you will have to provide the code for three members:

```csharp
// A getter-only property that will tell Duality approximately how big the 
// object is, so it knows when the Camera is far enough to just skip rendering 
// this object entirely.
// Based on Transform.Pos, what is the radius of the smallest circle that
// includes the entire visual representation of the object? If any part of that
// circle is on screen, the object will be rendered. Otherwise, Duality may skip
// rendering it.
float BoundRadius { get; }

// A method that checks whether this renderer is visible to the specified
// drawing device. This is where you will check if your renderer is near
// enough, and also if the device should see it at all.
bool IsVisible(IDrawDevice device);

// A method that uses the drawing device to actually draw the object.
void Draw(IDrawDevice device);
```

Let's say you're implementing a very simple renderer that draws a circle object with a fixed radius. It would look something like this:

```csharp
public class SimpleCircleRenderer : Component, ICmpRenderer
{
	private const float SampleCircleRadius = 50.0f;
	
	float ICmpRenderer.BoundRadius
	{
		get { return SampleCircleRadius; }
	}

	bool ICmpRenderer.IsVisible(IDrawDevice device)
	{
		bool anyGroupFlag = 
			(device.VisibilityMask & VisibilityFlag.AllGroups) 
			!= VisibilityFlag.None;
		bool screenOverlayFlag = 
			(device.VisibilityMask & VisibilityFlag.ScreenOverlay)
			!= VisibilityFlag.None;
		
		if (!anyGroupFlag) return false;
		if (screenOverlayFlag) return false;
		
		Vector3 myWorldPos = this.GameObj.Transform.Pos;
		return device.IsCoordInView(myWorldPos, SampleCircleRadius);
	}
	void ICmpRenderer.Draw(IDrawDevice device)
	{
		// Draw things!
	}
}
```

Now the `IsVisible` isn't as trivial as you thought it would be, right? Let me explain how it works.

First of all, it is important to keep in mind that any renderer could get drawn multiple times per frame - if there is more than one Camera, for example, the same object might appear in different views and be seen from different points in space. Every Camera will ask all renderers if it can or should see them. The same is true if a Camera has multiple rendering passes where one pass might see an object and another might not: `IsVisible` will be called for each pass that _might_ see it.

Every Camera and rendering pass has a bitmask that specifies which groups of objects it can see. These groups are for the most part user-defined and by default, all the objects belong to the first group and all Cameras and passes can see all groups. Each group has its own `VisibilityFlag` item that is set or not set in the `device.VisibilityMask` bitmask, depending on whether a Camera or rendering pass can see it.

```csharp
bool anyGroupFlag = 
	(device.VisibilityMask & VisibilityFlag.AllGroups)
	!= VisibilityFlag.None;
// ...
if (!anyGroupFlag) return false;
```

There are up to 31 visibility groups and for our sample above, our renderer doesn't really care and thus will assume to belong to all of them. As a result, it will be fine with any of the group flags being set, but return `false` if the drawing device can't see _any_ of the existing visibility groups.

```csharp
bool screenOverlayFlag = 
	(device.VisibilityMask & VisibilityFlag.ScreenOverlay) 
	!= VisibilityFlag.None;
// ...
if (screenOverlayFlag) return false;
```

One of those flags is not a group flag and has a special role though. In Duality, a Camera can render an object either in world space, or in screen space. A world space rendering will not follow the Camera and remain in the same spot on- or off screen while the Camera moves around. On the other hand, a screen space rendering is like drawing directly on the Camera lens - it doesn't matter where the Camera is or what it is looking at, that drawing will always be in the same spot on the recording.

By default, a Camera has two rendering passes: One that draws all world space objects and one that draws all screen space objects on top of them. Since Duality doesn't know which object belongs to which, it will ask all renderers whether they are visible in each pass. Depending on what you want, `IsVisible` needs to return `true` for one of them and `false` for the other. The `VisiblityFlag.ScreenOverlay` is dedicated to identifying a rendering pass that renders in screen space: Your renderer can simply check if the flag is either set or not set and decide in which of the two cases to be visible.

```csharp
Vector3 myWorldPos = this.gameobj.Transform.Pos;
return device.IsCoordInView(myWorldPos, SampleCircleRadius);
```

Last, there is a simple visibility check: Assuming a circle with our bounding radius, could any of it be inside the viewing volume of the Camera that is asking? If yes, the object should be rendered. Otherwise, we can return `false` to avoid rendering it entirely for this frame and improve performance.

## Option B: Deriving From `Renderer`

When writing a custom renderer that does the usual and simply renders a world space object, we can avoid duplicating some of the code that you saw in the `ICmpRenderer` version above. Instead of implementing the interface directly, derive your Component class from the abstract `Renderer` Component that [does some of the boilerplate things](https://github.com/AdamsLair/duality/blob/master/Source/Core/Duality/Components/Renderer.cs) for you. Our simplified implementation of the circle renderer would look like this:

```csharp
public class SimpleCircleRenderer : Renderer
{
	private const float SampleCircleRadius = 50.0f;
	
	public override float BoundRadius
	{
		get { return SampleCircleRadius; }
	}

	public override void Draw(IDrawDevice device)
	{
		// Draw things!
	}
}
```

As a bonus, the base class also allows to assign your custom renderer to any of the visibility groups in the editor. Simple, isn't it?

# Implementing Drawing Code

In a somewhat similar way, we have two approaches for implementing our drawing code: A very high-level one using the `Canvas` helper class, and the bottom-line by talking to `IDrawDevice` directly.

## Using the `Canvas` Class

The `Canvas` class allows us to draw geometric shapes easily without worrying too much about the details. Our circle renderer could look like this:

```csharp
public override void Draw(IDrawDevice device)
{
	// Prepare a Canvas object to make drawing easier
	Canvas canvas = new Canvas(device);
	
	// Where are we in world space?
	Vector3 pos = this.GameObj.Transform.Pos;
	
	// Draw things!
	canvas.State.ColorTint = ColorRgba.Green;
	canvas.FillCircle(pos.X, pos.Y, pos.Z, SampleCircleRadius);
}
```

If you want to slap a texture on that circle, render it additively or with a special shader, use `canvas.State.SetMaterial`. You'll also find some other properties that will allow you to specify additional parameters, like depth offset, the applied region of the texture, Font to use for text rendering and so on. Other parameters allow you to transform the rendered shapes, so you can rotate or scale what is rendered easily.

## Using the `IDrawDevice` Interface

If you have a great deal of things to render (like particles in a particle effect, or tiles in a tile map) using the high-level `Canvas` helper can get in the way of maximum performance. In other cases, `Canvas` might now provide what you need. Fortunately you can also talk directly to the drawing device and submit so-called "drawing batches".

Each drawing batch consists of a set of vertices that define _which_ geometry to draw, as well as a Material that represents _how_ that geometry will be rendered. The vertices you submit need to be expressed relative to the Camera position and already scaled to apply the parallax effect based on Z distance - which you'll have to take care of manually.

```csharp
public override void Draw(IDrawDevice device)
{
	// Determine relative position and scale of the object based on
	// where the Camera is and whether a parallax effect is applied.
	Vector3 posTemp = this.gameobj.Transform.Pos;
	float scaleTemp = this.gameobj.Transform.Scale;
	device.PreprocessCoords(ref posTemp, ref scaleTemp);

	// Display a centered [50, 50] rectangle
	Rect rectTemp = new Rect(-25, -25, 50, 50);

	// Create an array containing the four vertices of our single quad
	VertexC1P3T2[] vertices = new VertexC1P3T2[4];

	// Define the first vertex
	vertices[0].Pos.X = posTemp.X + rectTemp.TopLeft.X * scaleTemp;
	vertices[0].Pos.Y = posTemp.Y + rectTemp.TopLeft.Y * scaleTemp;
	vertices[0].Pos.Z = posTemp.Z;
	vertices[0].Color = ColorRgba.Red;
	
	// ... define the other three vertices ...

	// Submit a drawing batch
	device.AddVertices(this.sharedMaterial, VertexMode.Quads, vertices);
}
```

The power of this approach is in how pure it is: At its core, you only write data into an array. This allows you to perform a lot of optimizations when it comes to efficiently rendering thousands of tiles or particles - at the cost of convenience. In a lot of cases, the `Canvas` approach above is sufficient and easier to handle though and most of the time you don't need to go that extra mile.

For a more complex and versatile implementation sample of a renderer Component that uses `IDrawDevice` directly, take a look at the [SpriteRenderer](https://github.com/AdamsLair/duality/blob/master/Source/Core/Duality/Components/Renderers/SpriteRenderer.cs) implementation.

# Example: A Custom HUD Renderer

The following example simply renders the mouse cursor and its position as part of a HUD. Note that, unlike the previous renderer samples, it operates in screen space and uses screen coordinates.

```csharp
public class HudRenderer : Component, ICmpRenderer
{
	private ContentRef<Font> font = null;
	[DontSerialize] private CanvasBuffer buffer = null;

	public ContentRef<Font> Font
	{
		get { return this.font; }
		set { this.font = value; }
	}
	float ICmpRenderer.BoundRadius
	{
		get { return float.MaxValue; }
	}

	bool ICmpRenderer.IsVisible(IDrawDevice device)
	{
		// Only render when in screen overlay mode and the visibility mask is non-empty.
		return 
			(device.VisibilityMask & VisibilityFlag.AllGroups) != VisibilityFlag.None &&
			(device.VisibilityMask & VisibilityFlag.ScreenOverlay) != VisibilityFlag.None;
	}
	void ICmpRenderer.Draw(IDrawDevice device)
	{
		// Create a buffer to cache and re-use vertex arrays. Not required, but will boost performance.
		if (this.buffer == null) this.buffer = new CanvasBuffer();

		// Create a Canvas to auto-generate vertices from high-level drawing commands.
		Canvas canvas = new Canvas(device, this.buffer);
		canvas.State.SetMaterial(new BatchInfo(DrawTechnique.Alpha, ColorRgba.White));
		canvas.State.ColorTint = ColorRgba.Green.WithAlpha(0.5f);
		canvas.State.TextFont = this.font;

		// Display a mouse cursor as a simple filled circle
		canvas.FillCircle(DualityApp.Mouse.X, DualityApp.Mouse.Y, 5.0f);
		
		// Draw some text next to the cursor
		string cursorText = string.Format("{0}, {1}", (int)DualityApp.Mouse.X, (int)DualityApp.Mouse.Y);
		canvas.DrawText(cursorText, DualityApp.Mouse.X, DualityApp.Mouse.Y);
	}
}
```

For a real-world example, take a look at the [HUD renderer](https://github.com/AdamsLair/duality/blob/master/Samples/DualStickSpaceShooter/Gameplay/HeadUpDisplay.cs) from the space shooter, which you can find among the sample packages.