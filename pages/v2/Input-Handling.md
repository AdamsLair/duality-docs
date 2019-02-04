---
title: "Input Handling"
category: "basics"
displayOrder: 20
version: "v2"
---

This article will provide a quick overview on how to manage user input in Duality.

# Getting User Input

In Duality, user input comes in one of four different forms, depending on the input device you're querying: Mouse, keyboard, gamepad and joystick input are supported. All the input devices are accessible using public, static `DualityApp` API and are identified by a description string that they provide. They may become available or unavailable at runtime.

To get a quick overview, you can also check out the **Input Handling sample** in the package manager. It demonstrates how to use ever input device and as all samples, it comes with source code.

## Mouse

Mouse input is available via `DualityApp.Mouse` and provides the usual information with its `X`, `Y` and `Wheel` properties. For each of them, you can also query a `*Speed` value that represents the difference of the value since last frame. For position, there is also `Pos` and `Vel` as a vector-based shorthand.

```csharp
// Drawing the cursor using a screen space renderer Component
Vector2 cursorPos = DualityApp.Mouse.Pos;
canvas.DrawCircle(cursorPos.X, cursorPos.Y, 3);
```

All positions that are provided are relative to the game window, so `(0, 0)` will always be the window's upper left, regardless of the game window's position on screen. Note that the `X` and `Y` properties are assignable as well, which can be used to trap the mouse cursor and provide continuous mouse movement as long as the game window has input focus. You can also check whether the cursor is currently hovering the game window using the `IsAvailable` property.

To check whether a mouse button is currently pressed, either use the `MouseButton` indexer, or the `ButtonPressed` method. When you're interested in a one-time hit of a button, use the `ButtonHit` method instead.

```csharp
bool isPressed = DualityApp.Mouse[MouseButton.Left];
```

```csharp
bool isPressed = DualityApp.Mouse.ButtonPressed(MouseButton.Left);
bool justHit   = DualityApp.Mouse.ButtonHit(MouseButton.Left);
```

Besides polling mouse state every frame, you can also subscribe to events to be notified when mouse state changes. Note that you will have to unsubscribe your event handlers again when using events. There is no significant performance advantage of one way or another, so use whatever fits your design best.

## Keyboard

Keyboard input is available via `DualityApp.Keyboard` and it provides information about two things: Pressing keys, and entering text. While the two might seem very identical on a surface level, they do differ quite a bit when it comes to keyboard layout schemes and special character mapping.

Checking whether a certain key is pressed works the same as with retrieving mouse button input:

```csharp
bool isPressed = DualityApp.Keyboard[Key.Escape];
```

```csharp
bool isPressed = DualityApp.Keyboard.KeyPressed(Key.Escape);
bool justHit   = DualityApp.Keyboard.KeyHit(Key.Escape);
```

State change events for keys being pressed or released are available as well, with the same rules applying.

Retrieving text that was entered by the user can be done by checking the `CharInput` property for a non-empty string, and joining it with previously entered text. The user's operating system settings and keyboard layout schemes are applied in the background, so you don't have to care about any of this when retrieving text.

Key checks, on the other hand, are roughly based on [Scan Codes](https://en.wikipedia.org/wiki/Scancode) and thus are tied to an approximate physical location on the keyboard, rather than the symbol that is written on it. Based on a standard american [QWERTY keyboard](https://en.wikipedia.org/wiki/QWERTY), `Key.Y` for example will always be the one "next to T", regardless of whether it's actually a Y, Z, or something else. As a consequence, your "WASD" input will work in any typical keyboard layout, while looking like "WASD" on a QWERTY keyboard and "ZQSD" on an AZERTY keyboard. _If you're interested, you can read more about the reasoning behind this [here](https://github.com/opentk/opentk/pull/269#issuecomment-184367936)._

Like the mouse input, keyboard input can become available and unavailable based on window state: It's available as long as the game window has focus, and unavailable otherwise.

## Gamepads

Gamepad input retrieval works very similar to keyboard and mouse and doesn't really require much more explanation. They provide API for retrieving both buttons and continuous axes, as well as lots of specialized properties and methods to access thumbsticks, triggers and DPad explicitly. The biggest difference to the previously described input methods is that there can be any number of gamepads, which you can select via index or description string.

```csharp
Vector2 leftStick = DualityApp.Gamepads[0].LeftThumbstick;
```

Note that you can access any gamepad index or description, even if no such gamepad has been detected yet - non-existent or currently disconnected gamepads will report to be unavailable. You can enumerate all gamepads that have been detected at some point by checking `Gamepads.Count` or iterating over `Gamepads` using a `foreach` loop.

## Joysticks

Gamepads are a specialized form of joystick, so every gamepad input will also show up as a joystick input. In addition, `DualityApp.Joysticks` will list all user input devices that are not mouse or keyboard.

Handling and public API surface are similar to what was described in the Gamepads section, and need not be repeated. What's different is that there are no specialized properties and methods for "DPads" or "triggers", because we do not have any information about the actual shape of the input device. Instead, there are generic buttons, axes and hats, which can be queried. Each of them has a distinct `Count` property, so it is possible to detect a limited set of joystick capabilities as well.

# Internals and Advanced Input Management

Duality's input system is designed in two layers: The upper "state manager" layer, and the lower "input source" layer. In your typical game, you only interact with state managers, but when writing custom launchers, backends, debugging utility or simulated input, you may be interested in the "input source" layer as well.

## State Managers

A state manager is a mid-level construct where your game asks for user input. All previously discussed `DualityApp` properties return state managers. They query the low-level user input from the backend, store it consistently for the entire simulation frame, keep track of changes and velocities, and emit events when you are subscribed to them.

They form an abstraction layer that hides away changes, so that a gamepad instance remains the same when unplugging and reconnecting the gamepad later. State managers also provide a fixed API surface which cannot be modified and does not provide support for overrides. You can, however, inject custom input sources.

## Input Sources

Unlike a state manager, an input source is a thin interface for querying device stated. Every state manager needs an input source: The standard sources are assigned automatically, but you can replace the source of any input device easily by assigning the `Source` property. You can also add more gamepads and joysticks by adding their sources using the respective `DualityApp` API: For every source that you add, a matching state manager will be automatically generated and available.

Input sources share the `IUserInputSource` interface, but otherwise [diverge in their API contract](https://github.com/AdamsLair/duality/tree/master/Source/Core/Duality/Input/Sources) depending on the requirements of the device type they're dealing with. They provide the means to query momentary state, but do not require memory or processing of any kind - that's what state manager will add on top.

# Custom Input Devices

You can provide custom input devices using the same input facilities that every Duality application uses by implementing the appropriate `IUserInputSource` interface and assigning an instance of your class to the `Source` parameter of the target state manager. In case of joystick or gamepad input, you can also use the `AddSource` and `RemoveSource` methods to generate new input devices based on your source implementation.

The manager / source abstraction is useful mainly in the context of both platform and execution environment abstraction, but you can also use it to provide an interface to completely custom input devices, or to record and replay, or even simulate user input.