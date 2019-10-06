---
title: "Common Pitfalls"
parent: "Guidelines"
displayOrder: 0
version: "v3"
---

Typical errors which can be easily avoided.

# Workflow

## Ignoring Logged Errors

Duality has a logging system that knows three basic types of logs: Infos, Warnings and Errors. While developing and testing games with Duality, you should be careful and keep an eye on the **Log View** in the editor, or the console window in your test runs.

* **Info** messages can safely be ignored, they're usually just providing context and tell you what happens.
* **Warning** messages are a reason for concern, but some of them are fine. They want you to look at them an investigate.
* **Error** messages in the log are **never** a good sign. Something is seriously wrong, and you should investigate immediately before continuing to work on your game. Even if the editor is able to recover from certain errors, others may lead to data loss or unexpected / erratic behavior when ignored. (_Hint:_ If you lost something, take a look at the `Backups` folder!)
