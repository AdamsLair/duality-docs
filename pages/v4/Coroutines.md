---
title: "Coroutines"
category: "basics"
displayOrder: 10
version: "v4"
---

First of all, what is a Coroutine?

Taken from Wikipedia:

_Coroutines are very similar to threads. However, coroutines are cooperatively multitasked, whereas threads are typically preemptively multitasked. This means that coroutines provide concurrency but not parallelism. The advantages of coroutines over threads are that they may be used in a hard-realtime context (switching between coroutines need not involve any system calls or any blocking calls whatsoever), there is no need for synchronisation primitives such as mutexes, semaphores, etc. in order to guard critical sections, and there is no need for support from the operating system._

A Coroutine is a piece of code that executes "in parallel", over a certain amount of time, or generally asynchronously, with another; in our particular case, it is code that should run for an arbitrarily long amount of frames, without getting in the way of the usual flow of your code.

This **does not mean** that the Coroutine is operating on a separate thread, and thus can be written ignoring the usual constraints of the Update-Draw loop of the game engine; each step of the Coroutine should be able to be performed in one Update call, so don't go around looking for prime numbers in a Coroutine, chances are your game would stutter.

# A simple example

Let's say, for example, that you want to code a Component that, upon pressure of a key, will make your GameObject's move 200 units to the right: with only this requirement you will probably end up with something like this

```csharp
public void OnUpdate()
{
	if (DualityApp.Keyboard.KeyHit(Key.Space))
	{
		this.GameObj.Transform.Pos.MoveBy(new Vector2(200, 0));
	}
}
```

Which is fine, and works well for you.
But now, let's assume you want the transition to be smooth; one way would be to move 1 unit per frame (let's ignore the time it would take or the fact that it would be dependent on the frame rate)

```csharp
private Vector2? movementTarget = null;
private Vector2 movementDirection = Vector2.Zero;

public void OnUpdate()
{
	if(movementTarget != null)
	{
		if((this.movementTarget.Value - this.GameObj.Transform.Pos.Xy).LengthSquared >= this.movementDirection.LengthSquared)
		{
			this.GameObj.Transform.Pos.MoveBy(movementDirection);
		}
		else
		{
			this.GameObj.Transform.Pos.MoveTo(this.movementTarget.Value);
			this.movementTarget = this.Game;
		}
	}
	else if (DualityApp.Keyboard.KeyHit(Key.Space))
	{
		this.movementTarget = this.GameObj.Transform.Pos.Xy + new Vector2(200, 0);
		this.movementDirection = (this.movementTarget.Value - this.GameObj.Transform.Pos.Xy).Normalized;
	}
}
```

Again, this works, but the OnUpdate method is starting to appear cluttered with unnecessary code; wouldn't it be great if the part of code in charge of managing the movement was in another part?
Sure, you could refactor the code to extract the method and have a `UpdateColor` or similar being called at the end or beginning of `OnUpdate`; good, but we can still do better. We can make a Coroutine out of it.

A Coroutine is implemented as a method that returns an `IEnumerable<WaitUntil>` collection, and whose body is separated in execution sections by `yield return` statements. At instantiation, and at each Update loop, the method is executed from its current point until the next `yield`, and so its total execution can be spread out over a number of frames.

To get back to our previous example, we could have our Coroutine as follows:

```csharp
private IEnumerable<WaitUntil> MoveTo(Vector2 target)
{
	Vector2 movement = (target - this.GameObj.Transform.Pos.Xy).Normalized;
	
	while((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
	{
		this.GameObj.Transform.Pos.MoveBy(movement);
		yield return WaitUntil.NextFrame;
	}
	
	this.GameObj.Transform.Pos.MoveTo(target);
}
```

Let's see it more in detail:

```csharp
private IEnumerable<WaitUntil> MoveTo(Vector2 target)
```

this is our Coroutine definition; note that, being a method like all others, it can have parameters and it can have any kind of access modifier, as long as it is accessible by its caller.

```csharp
Vector2 movement = (target - this.GameObj.Transform.Pos.Xy).Normalized;
```

We don't need anymore to keep a private class variable; now we can just keep our movement data inside the Coroutine and declutter the rest of the class from things that are used only in specific cases

```csharp
while((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
{
	this.GameObj.Transform.Pos.MoveBy(movement);
```

This part should be pretty simple: just keep on moving our GameObject's position until it's close enough to the target location, so close actually that if we were to move one more time we would go past it. But why doesn't this piece of code loop 200 times and move the GameObject to the target location instantaneously? Because of this little line here

```csharp
yield return WaitUntil.NextFrame;
```

This is where the magic happens: this line is what allows the Coroutine to relinquish control to the main Update loop, and it is from where it will continue its execution the next frame, when it will be advanced until it encounters the next yield instruction. 
This means that, being inside a loop, the code will run multiple times until the GameObject has finally moved close enough to its target location, and the loop will end.

So, to recap, this is what actually happens with the Coroutine:

```
Coroutine is instantiated, calculates the value for `movement`, enters the loop, moves the GameObject a bit, and yields execution until the next frame
an Update loop starts, the Coroutine resumes its execution, loops back, moves the GameObject a little more, and yields execution again
an Update loop starts, the Coroutine resumes its execution, loops back, moves the GameObject a little more, and yields execution again
...
an Update loop starts, the Coroutine resumes its execution, loops back, but this time the GameObject is finally close enough to its target, so it exits the while, moves the GameObject to its final target position, and ends.
```

And to put all this together in our example, 

```csharp
public void OnUpdate()
{
	if (DualityApp.Keyboard.KeyHit(Key.Space))
	{
		Vector2 target = this.GameObj.Transform.Pos.Xy + new Vector2(200, 0);
		this.Scene.StartCoroutine(this.MoveTo(target));
	}
}

private IEnumerable<WaitUntil> MoveTo(Vector2 target)
{
	Vector2 movement = (target - this.GameObj.Transform.Pos.Xy).Normalized;
	
	while((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
	{
		this.GameObj.Transform.Pos.MoveBy(movement);
		yield return WaitUntil.NextFrame;
	}
	
	this.GameObj.Transform.Pos.MoveTo(target);
}
```

Doesn't it look much better now? It might look a bit underwhelming but that's only because the coroutine-d method is quite simple, but try to imagine that you want your GameObject to move through a number of points, waiting half a second between each stop, how would your OnUpdate method look like?
Probably there would be need for some status variable, some tracking of on which leg of the path you currently are, tracking of time, and so on; with a Coroutine you can do something like this

```csharp
private IEnumerable<WaitUntil> MoveComplex(Vector2[] points, float waitingAtStop)
{
	foreach(Vector2 target in points)
	{
		Vector2 movement = (target - this.GameObj.Transform.Pos.Xy).Normalized;
		
		while((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
		{
			this.GameObj.Transform.Pos.MoveBy(movement);
			yield return WaitUntil.NextFrame;
		}
		
		this.GameObj.Transform.Pos.MoveTo(target);
		yield return WaitUntil.Seconds(waitingAtStop);
	}
}
```


# WaitUntil 

At the core of a Coroutine are lines of code interspaced by a bunch of `WaitUntil` yields.
They are the basis of the infrastructure that makes your Coroutine run together with the rest of your game, either frame-by-frame, as we saw before, or with arbitrary timings.

Your Coroutine can be made to suspend its execution for
* the next frame - WaitUntil.NextFrame
* a number of frames - WaitUntil.Frames(n)
* an amount of time - WaitUntil.Seconds(n) or WaitUntil.TimeSpan(ts), either in game (scaled) or real time
* until another coroutine has terminated its execution - WaitUntil.CoroutineEnds(anotherCoroutine)

# How long can a Coroutine run for?

Although there is no limit to the total duration of a Coroutine, which could make it an interesting choice for long-running subroutines, ideally that need to run with some timing, the Coroutines are **not** serialized and thus you can not expect to find them resuming from their last state when reloading the Scene. The status is maintained as long as the engine is running, so that it's possible to schedule a Coroutine on a different Scene, and have it start/resume as soon as the relevant Scene becomes active again.