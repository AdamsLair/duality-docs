---
title: "Coroutines"
category: "basics"
displayOrder: 10
version: "v4"
---

First of all, what is a Coroutine?

Coroutines are very similar to threads, in the sense that they allow multiple branches of code to advance "in parallel", with the difference that actually there are no threads; instead, a Coroutine shares its execution time with the rest of the code.
A Coroutine, logically speaking, is a method that executes for a while, stops, and can resume its execution from the place it was the next time it is executed, and it can be considered similar in its evolution to a Finite State Machine.
A Coroutine is scheduled for execution on a desired Scene, is executed inside the Scene's `Update` method, and is usually brought to completion over the span of multiple update calls. 

For this reason, you need to be mindful of the usual constraints of the Update-Draw loop of the game engine; each step of the Coroutine should be able to be performed in one Update call, so don't go around looking for prime numbers in a Coroutine, as chances are your game will stutter.

Given their time-based nature, Coroutines are well suited for dealing with animations, effects, and game-entity-related behavior code

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
	if (movementTarget != null)
	{
		if ((this.movementTarget.Value - this.GameObj.Transform.Pos.Xy).LengthSquared >= this.movementDirection.LengthSquared)
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
Sure, you could refactor the code to extract the method and have a `UpdatePosition` or similar being called at the end or beginning of `OnUpdate`; good, but we can still do better. We can make a Coroutine out of it.

A Coroutine is implemented as a method that returns an `IEnumerable<WaitUntil>` collection, and whose body is logically separated in execution sections by `yield return` statements. At instantiation, and at each Update loop, the method is executed from its current point until the next `yield`, and so its total execution can be spread out over a number of frames.

To get back to our previous example, we could have our Coroutine as follows:

```csharp
private IEnumerable<WaitUntil> MoveTo(Vector2 target)
{
	Vector2 movement = (target - this.GameObj.Transform.Pos.Xy).Normalized;
	
	while ((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
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

this is our Coroutine signature; note that, being a method like all others, it can have parameters and it can have any kind of access modifier, as long as it is accessible by its caller.

```csharp
Vector2 movement = (target - this.GameObj.Transform.Pos.Xy).Normalized;
```

We don't need anymore to keep a private class variable; now we can just keep our movement data inside the Coroutine and declutter the rest of the class from things that are used only in specific cases

```csharp
while ((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
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
	
	while ((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
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
	foreach (Vector2 target in points)
	{
		Vector2 movement = (target - this.GameObj.Transform.Pos.Xy).Normalized;
		
		while ((target - this.GameObj.Transform.Pos.Xy).LengthSquared >= movement.LengthSquared)
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
* the next frame - `WaitUntil.NextFrame`
* a number of frames - `WaitUntil.Frames(n)`
* an amount of time - `WaitUntil.Seconds(n)` or `WaitUntil.TimeSpan(ts)`, either in game (scaled) or real time

# How long can a Coroutine run for?

Although there is no limit to the total duration of a Coroutine, keep in mind that Coroutines are **not** serialized and thus you can not expect to find them resuming from their last state when reloading the Scene. The state is maintained as long as the engine is running, so that it's possible to schedule a Coroutine on a different Scene, and have it start/resume as soon as the relevant Scene becomes active again, but nothing more.
Of course this means you can have an endless Coroutine as well, simply have a while (true) inside its implementation that continuosuly yields but never actually ends. The only way to stop this kind of Coroutine would be to `Cancel` it.

# Managing Coroutines

Another interesting thing you can do with Coroutines, is to control their execution: we already said that a Coroutine is scheduled on a specific Scene. In particular, through this line here

```csharp
this.Scene.StartCoroutine(this.MoveTo(target));
```

we see that the Coroutine was scheduled on the current Scene returned by our Component; of course you can start a Coroutine on any Scene you can reference or obtain throuhg the ContentProvider, just remember that, as we said at the beginning, ti won't be actually executed until said Scene updates.
If we were to change the line to

```csharp
this.moveCoroutine = this.Scene.StartCoroutine(this.MoveTo(target));
```

we could `Pause()`, `Resume()` or `Cancel()` it as necessary, for example because the spacebar has been pressed again and we want to restart the whole movement from the current point, in which case it would be as easy as doing

```csharp
this.moveCoroutine?.Cancel();
this.moveCoroutine = this.Scene.StartCoroutine(this.MoveTo(target));
```

Another important point to keep in mind is that a Coroutine has no concept of "owner" GameObject or Component by itself, and as such it will continue its execution even if the generating entity has been removed from the Scene. It will be your task to manage the situation accordingly, if your code requires it.
In any case, a Coroutine encountering an unhandled Exception will immediately stop its execution and will be de-scheduled from the next Update.

# Multiple Coroutines

As we saw, a Coroutine can be a powerful tool by itself, but this power can increase by using multiple Coroutines together, either in parallel, or by integrating one inside another.

Imagine to have prepared Coroutines to manage an enemy's behavior (kind of a rudimentary AI): 
* `Idle()`, endless, which makes the enemy do nothing
* `Scan(frequency)`, endless, which makes the enemy look left and right with a certain frequency
* `Patrol()`, endless, which makes the enemy follow a list of points
* `Focus(point, time)`, timed, which makes the enemy look at a specific point for a certain amount of time

Given these methods, we can create complex behaviors like this (only Coroutine management is covered)

```csharp
private List<Coroutine> activeCoroutines;

private void ClearCoroutines()
{
	foreach (Coroutine c in this.activeCoroutines)
		c.Cancel();
		
	this.activeCoroutines.Clear();
}

public void OnUpdate()
{
	bool notIdle = false;
	foreach (Coroutine c in this.activeCoroutines)
		notIdle |= c.IsAlive;

	if (!notIdle)
	{
		ClearCoroutines();
		this.activeCoroutines.Add(this.Patrol()); // [1]
		this.activeCoroutines.Add(this.Scan(...));
	}

	Vector2? noiseSource = CheckForNoises();
	Player player = CheckForEnemiesInSight();
	
	if (enemy != null)
	{
		ClearCoroutines();
		Coroutine c = this.Scene.StartCoroutine(this.Engage(enemy));
		this.activeCoroutines.Add(c);
	}
	else if (noiseSource.HasValue)
	{
		ClearCoroutines();
		Coroutine c = this.Scene.StartCoroutine(this.CheckLocation(noiseSource.Value));
		this.activeCoroutines.Add(c);
	}
}

private IEnumerable<WaitUntil> CheckLocation(Vector2 target)
{
	float checkingDistanceSquared = ...;
	Transform t = this.GameObj.Transform;
	
	while ((target - t.Pos.Xy).LengthSquared >= checkingDistanceSquared) // maybe also need line of sight
	{
		Vector2 movement = ...; // navigate around level
		
		t.Pos.MoveBy(movement);
		t.Angle = // face towards movement
		yield return WaitUntil.NextFrame;
	}
	
	Coroutine scan = this.Scene.StartCoroutine(this.Scan(...)); // [2]
	yield return WaitUntil.Seconds(2);
	scan.Cancel();
}

private IEnumerable<WaitUntil> Engage(Player target)
{
	float engagementDistanceSquared = ...;
	Transform t = this.GameObj.Transform;
	Transform e = target.GameObj.Transform;
	
	// display "!" icon on top of enemy
	
	foreach (WaitUntil waitCondition in this.Focus(target.Transform.Pos.Xy, .5)) // [3]
		yield return waitCondition;
	
	while (CheckForEnemiesInSight() == target) // check line of sight
	{
		Vector2 distance = (e.Pos.Xy - t.Pos.Xy).LengthSquared; 
		if (distance >= engagementDistanceSquared)
		{
			// move towards target
		}
		else
		{
			// shoot at target
		}
		yield return WaitUntil.NextFrame;
	}
	
	Coroutine scan = this.Scene.StartCoroutine(this.Scan(...)); // [2]
	yield return WaitUntil.Seconds(2);
	scan.Cancel();
}
```

Point `[1]` is the simplest way to combine multiple behaviors: in our case, our character will continuously move around its set patrol path (Patrol) while checking left and right (Scan)
In point `[2]` you can see one method to combine a Coroutine inside another: by instancing it and waiting for a certain amount of time, we can effectively pause the current Coroutine and let the other one run in its stead. Notice that we wait for a couple of seconds, and then `Cancel` the inner Coroutine because `Scan`, as we said, would never stop on its own.
Finally, in case your inner Coroutine can end, you can use point `[3]`, where we simply yield the current status of the inner Coroutine, which, in this case, is not updated directly by the CoroutineManager associated by the Scene, but as a side effect of the update performed on the main Coroutine containing it (hopefully you will never have to worry about this detail).
