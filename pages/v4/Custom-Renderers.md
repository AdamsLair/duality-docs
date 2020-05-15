---
title: "Custom Renderers"
category: "advanced"
displayOrder: 0
version: "v4"
---

This article will introduce you to writing custom renderer Components.

# What is a "Renderer Component"?

When you run your game, the things you see on screen are a visual representation of the objects you defined and created both in the editor and code. While Duality provides the facilities to talk to the render pipeline and ultimately draw things on the screen, it is the job of a renderer Component to _use_ those facilities and actually draw something. 

Depending on how you implement a renderer Component, it can display just about anything: A sprite, geometric shapes, a particle system, a whole tilemap, and so on. It may render in world space or screen space and it is not restricted to the concept of a single GameObject, or object at all.

# Defining a Renderer Component

The bottom line of creating a custom renderer is implementing the `ICmpRenderer` interface. Alternatively, you can derive from `Renderer`, which is an abstract Component type that implements some of the boilerplate code you need for your usual world-space object rendering.

In both cases, you will have to talk to the `IDrawDevice` interface - it's the API that Duality provides to let you draw things.

## Option A: Implementing `ICmpRenderer`

Choosing the bottom line approach, implementing the `ICmpRenderer` interface means you will have to provide the code for two members:

```csharp
// A getter method that populates a CullingInfo struct with the objects
// position, radius and visibility group flags. Once retrieved, this
// information is used by Duality to determine which renderers are visible
// to any particular camera or drawing device.
void GetCullingInfo(out CullingInfo info);

// A method that uses the drawing device to actually draw the object.
void Draw(IDrawDevice device);
```

Let's say you're implementing a very simple renderer that draws a circle object with a fixed radius. It would look something like this:

```csharp
public class SimpleCircleRenderer : Component, ICmpRenderer
{
	private const float SampleCircleRadius = 50.0f;
	
	void ICmpRenderer.GetCullingInfo(out CullingInfo info)
	{
		info.Position = this.GameObj.Transform.Pos;
		info.Radius = this.SampleCircleRadius;
		info.Visibility = VisibilityFlag.Group0;
	}
	void ICmpRenderer.Draw(IDrawDevice device)
	{
		// Draw things!
	}
}
```

Now, some background on visibility group flags: Every Camera and rendering pass has a bitmask that specifies which groups of objects it can see. These groups are for the most part user-defined and by default, all the objects belong to the first group and all Cameras and passes can see all groups. Each group has its own `VisibilityFlag` item that is set or not set in the `device.VisibilityMask` bitmask, depending on whether a Camera or rendering pass can see it.

There are up to 31 visibility groups and for our sample above, our renderer just assumes to be part of the default group zero. It will be visible to all cameras and drawing devices that can see this group, which, by default, is all of them.

One of the available visibility flags is not a group flag and has a special role though. In Duality, a Camera can render an object either in world space, or in screen space. A world space rendering will not follow the Camera and remain in the same spot on- or off screen while the Camera moves around. On the other hand, a screen space rendering is like drawing directly on the Camera lens - it doesn't matter where the Camera is or what it is looking at, that drawing will always be in the same spot on the screen.

The default rendering setup has two rendering passes: One that draws all world space objects and one that draws all screen space objects on top of them. Since Duality doesn't know which object belongs to which, it use each renderers culling info to determine whether they are visible in each pass. The `VisiblityFlag.ScreenOverlay` is dedicated to identifying a rendering pass that renders in screen space: Your renderer can simply set the flag when it belongs to screen space, or not set it when it's bound to world space.

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
[DontSerialize]
private Canvas canvas = new Canvas();

public override void Draw(IDrawDevice device)
{
	// Prepare the Canvas for rendering to the target device
	this.canvas.Begin(device);
	
	// Where are we in world space?
	Vector3 pos = this.GameObj.Transform.Pos;
	
	// Draw things!
	canvas.State.ColorTint = ColorRgba.Green;
	canvas.FillCircle(pos.X, pos.Y, pos.Z, SampleCircleRadius);

	// Finalize rendering with our Canvas
	this.canvas.End();
}
```

If you want to slap a texture on that circle, render it additively or with a special shader, use `canvas.State.SetMaterial`. You'll also find some other properties that will allow you to specify additional parameters, like depth offset, the applied region of the texture, Font to use for text rendering and so on. Other parameters allow you to transform the rendered shapes, so you can rotate or scale what is rendered easily.

## Using the `IDrawDevice` Interface

If you have a great deal of things to render (like particles in a particle effect, or tiles in a tile map) using the high-level `Canvas` helper can get in the way of maximum performance. In other cases, `Canvas` might not provide the functionality you need. Fortunately you can also talk directly to the drawing device and submit so-called "drawing batches".

Each drawing batch consists of a set of vertices that define _which_ geometry to draw, as well as a Material that represents _how_ that geometry will be rendered. The vertices you submit need to be expressed in world space, as they are automatically transformed as needed by the vertex shader later on.

```csharp
public override void Draw(IDrawDevice device)
{
	// Retrieve position and scale of the object from its Transform component.
	// We do not take into account rotation here, but we could, if needed.
	Vector3 pos = this.GameObj.Transform.Pos;
	float scale = this.GameObj.Transform.Scale;

	// Display a centered [50, 50] rectangle
	Rect rectTemp = new Rect(-25, -25, 50, 50);

	// Create an array containing the four vertices of our single quad
	VertexC1P3T2[] vertices = new VertexC1P3T2[4];

	// Define the first vertex
	vertices[0].Pos.X = pos.X + rectTemp.TopLeft.X * scale;
	vertices[0].Pos.Y = pos.Y + rectTemp.TopLeft.Y * scale;
	vertices[0].Pos.Z = pos.Z;
	vertices[0].Color = ColorRgba.Red;

	// Define the second vertex
	vertices[1].Pos.X = pos.X + rectTemp.TopRight.X * scale;
	vertices[1].Pos.Y = pos.Y + rectTemp.TopRight.Y * scale;
	vertices[1].Pos.Z = pos.Z;
	vertices[1].Color = ColorRgba.Red;
	
	// ... define the other two vertices ...

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
	[DontSerialize] private Canvas canvas = new Canvas();

	public ContentRef<Font> Font
	{
		get { return this.font; }
		set { this.font = value; }
	}

	void ICmpRenderer.GetCullingInfo(out CullingInfo info)
	{
		info.Position = Vector3.Zero;
		info.Radius = float.MaxValue;
		info.Visibility = VisibilityFlag.AllGroups | VisibilityFlag.ScreenOverlay;
	}
	void ICmpRenderer.Draw(IDrawDevice device)
	{
		this.canvas.Begin(device);

		// Set up the canvas as needed for our HUD
		canvas.State.SetMaterial(DrawTechnique.Alpha);
		canvas.State.ColorTint = ColorRgba.Green.WithAlpha(0.5f);
		canvas.State.TextFont = this.font;

		// Display a mouse cursor as a simple filled circle
		canvas.FillCircle(DualityApp.Mouse.Pos.X, DualityApp.Mouse.Pos.Y, 5.0f);
		
		// Draw some text next to the cursor
		string cursorText = string.Format("{0}, {1}", (int)DualityApp.Mouse.Pos.X, (int)DualityApp.Mouse.Pos.Y);
		canvas.DrawText(cursorText, DualityApp.Mouse.Pos.X, DualityApp.Mouse.Pos.Y);

		this.canvas.End();
	}
}
```

For a real-world example, take a look at the [HUD renderer](https://github.com/AdamsLair/duality/blob/master/Samples/DualStickSpaceShooter/Gameplay/HeadUpDisplay.cs) from the space shooter, which you can find among the sample packages.