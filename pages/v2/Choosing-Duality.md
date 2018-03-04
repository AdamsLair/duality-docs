---
title: "Should You Use Duality?"
category: "introduction"
displayOrder: 10
version: "v2"
---

Someone once said: "Every game engine sucks - but each in its own, unique way". Choosing an engine is not about going through a shopping list of billboard features, it's not even about deciding which one is "better". It's about setting priorities and identifying requirements: What does your game need, and what do _you_ need, as a developer? What do you value? This guide is here to help you decide whether your next project should be made with Duality.

_If you haven't read the info page yet, [head over here](http://duality.adamslair.net) first._

# About Duality

So let's take a close, honest look. If you're with us by the end of this chapter, Duality and you have a solid chance.

## Where it sucks

- **Its tooling is Windows-only**, so you need a Windows dev machine.
- **It doesn't export to a lot of platforms**, in fact the only one that's truly battle-tested is Windows. Duality has been reported to run on [Linux](https://github.com/AdamsLair/duality/issues/288) and [MacOS](https://github.com/AdamsLair/duality/issues/287), but there's no official support on that yet. And mobile platforms are off limits for now, unless you are ready to put in some serious effort.
- **Limited out-of-the-box capabilities** - there is, for example, no builtin UI system or multiplayer netcode. Be prepared to write code that you will rely on. Scripting together pre-built stuff is not going to cut it.
- **It's the road less traveled by**. We are not Unity, Cry or Unreal. There is uncharted territory and, when faced with a very specific problem, you may be the first one to solve it in our entire community.

## Where it's allright

- **A nice Editor** that doesn't have every function ever, but focuses on a small set of polished modules. User experience is a priority and we're continuously working on making things a little nicer.
- **A reasonable API** that does have some pain points, but generally favors sustainable and well-engineered solutions over quickfix scripting. You are in control if you want to be, and overall understanding is favored over magic.

## Where it excels

- **Open Source!** No really, I mean it. This is not a philosophical or idealistic argument, it's a practical advantage. You can read Duality code to see how everything works internally and truly understand what's happening behind the curtain. There is no such thing as an unsolvable problem, simply because you have all the source code. The sky is the limit.
- **An open architecture** allows you to extend both core and editor even without touching any of its source code. Duality plugins are not an afterthought, but ingrained in all of its APIs. It doesn't stop at custom Components or editor windows: Want to add a custom asset importer? Switch to a new serializer? Replace the entire OpenGL backend with a Vulkan one? Do it!
- **Duality promotes ownership** of both its own technology and the parts you add - the ownership of a craftsman who knows every inch of his house, because he was there when it was built and took part in it. The goal of Duality is not to do everything you _might_ want to do one day, but to provide you with the tools to do it yourself when the day comes.
- **It's all pure .Net**, so use whichever C#, F# or other .Net language you want. Duality does not tie you to a specific runtime environment and relies on well-tested common infrastructure rather than re-inventing it. Game code are just .Net assemblies, and package management is just NuGet in a wrapper - the ecosystem you already know.
- **Zero Licensing Issues**. It's just [plain old MIT](https://tldrlegal.com/license/mit-license). No cost involved, no splash screens or pro versions or subscription plans. 

Any questions? [Talk to us](http://forum.adamslair.net/)!