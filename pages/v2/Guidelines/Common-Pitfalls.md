---
title: "Common Pitfalls"
parent: "/pages/v2/Guidelines"
displayOrder: 0
version: "v2"
---

Typical errors that can be easily avoided.

# Coding

## Not checking `InitContext` in `OnInit`

The `ICmpInitializable` interface defines an `OnInit` method, which you can override to equip your `Component` with custom initialization behavior. However, many users forget to check for the `context` parameter that is provided! `OnInit` being called tells you nothing about _what_ kind of initialization is happening: Is the Component being activated? Loaded? Has it just been added to its GameObject for the first time? If you're doing something regardless of context, you will react to **all** of these contexts, which is almost certainly not what you wanted. In most cases reacting to `InitContext.Activate` is what's actually needed.

# Workflow

## Ignoring Logged Errors

Duality has a logging system that knows three basic types of logs: Infos, Warnings and Errors. While developing and testing games with Duality, you should always keep an eye on the **Log View** in the editor, or the console window in your test runs.

* **Info** messages can safely be ignored, they're usually just providing context and tell you what happens.
* **Warning** messages are a reason for concern, but some of them are fine. They want you to look at them an investigate.
* **Error** messages in the log are **never** a good sign. Something is seriously wrong, and you should investigate immediately before continuing to work on your game. Even if the editor is able to recover from certain errors, others may lead to data loss or unexpected / erratic behavior when ignored. (_Hint:_ If you lost something, take a look at the `Backups` folder!)