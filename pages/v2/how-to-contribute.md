---
category: "Contributing"
title: "How to Contribute"
---

# Introduction

First of all: Thanks for your willingness to contribute to Duality. It has grown to be a big project and one developer alone can't possibly handle all the issues at once. 

That said, external contributions are always a double-edged sword, as they introduce a little extra work due to their necessity to be reviewed and potentially edited or merged to fit within the existing framework of the project. This guide aims to help you minimize this extra effort and make your contributions as helpful as possible.

# Ways to contribute

There are many ways to contribute, some of them disjoint and others heavily integrated into the project. Here's a quick overview on what you can do, sorted by their difficulty level and the amount of required experience with the project.

## Step One: Be a Community Wizard

This task is probably the one best fitted for new users: If you didn't do anything with Duality, or just recently discovered it but still want to contribute, this is for you.

As cool as Duality might be, let's not deceive ourselves here: It's still the new guy in gamedev town and a lot of older inhabitants prefer to curiously look at it from a distance rather than walk over and say hello. Those brave souls who jump right in and use it for their projects are rare, but maybe the most important contributors there are right now. If you're reading this, you are probably one of them, and let me tell you this: Duality needs you.

The most important tools of the trade are [Duality itself](http://duality.adamslair.net), the [Forum](http://forum.adamslair.net) and any means you're personally using to publish things on the internet, say, a blog or similar. Let's see what you can do:

1. **Learn how to use Duality** and ask clever questions. You may be still new, but you can nevertheless be inspiring to others by showing them what's important to you, or difficult to grasp at first.
2. **Develop some games** with Duality. Take part in Game Jams. Use it to create awesome stuff. Or silly stuff. Stuff that is just fun to make, nice to look at or even awesome in its own way.
3. **Help others** who are new to Duality or try to do something that you have already done. Answer their clever questions. Show them the way things are done, and ask what they're trying to do instead of telling them what they shouldn't. Be inspiring to them.
4. **Write tutorials**. Do a making of for YouTube. Blog about it. Enrich the community with inspiration and knowledge.

This might look like a whole lot of work, but really, if you're even starting to do one of the things, you are already a great help. Be friendly, honest and humble. Let's build a community together.

## Step Two: Be a Plugin Developer

This is one of the easier ways to help, and certainly a very independent one. Compared to big players like Unity, the Duality framework hasn't seen nearly as much "man hours" of work and thus lacks certain functionality that you might require or wish for. But thankfully, Duality is a very modular framework as well! There's nothing that stops you from implementing the functionality you need in a distinct Core- or EditorPlugin and then distributing that plugin for others to use. It might feel more like a side-project to Duality instead of actually contributing, but rest assured that this is a task just as vital and helpful as any other, and a lot of Duality's core functionality is implemented in plugins as well! 

So, how do you proceed? Well, this is mostly your own choice, because you're working in a distinct module that you can shape exactly the way you like. Here are some pointers, though:

1. Set up a GitHub repository for your module.
2. Set up a Duality plugin project / solution and push it. In case you need examples on how to do this, you might want to take a look at some minimal examples from the main repository. It's quite easy - just add a reference to the appropriate Duality Assembly and provide an implementation of the matching Plugin class.
3. Implement a first prototype.
4. Publish its binaries and [let others know](http://forum.adamslair.net).

## Step Three: Develop the Framework

Congratulations! You have picked the hardest part of them all, but nonetheless one with the highest potential to have a huge impact on all Duality developers, be it good or bad. What you are doing is introducing new features, fixes and concepts to the framework that powers it all. It is a very cautious work, and you'll probably have to talk to [the dev team](https://github.com/orgs/AdamsLair/people) a lot, and maybe curse them a bit for rejecting and editing your Pull Requests. We're sorry. But here comes the good part: It will be worth it when it actually works out.

Now, what do you actually do? Here's how to start:

1. **Fork** [Duality](https://github.com/AdamsLair/duality) on GitHub.
2. **Evaluate** about what exactly you want to do.
  * Fix a bug: This is probably the best option for starters. It's small, self-contained and doesn't introduce any new stuff.
  * Add a feature: This is a little harder. Although still small and self-contained, it extends the concept space of Duality and you will need to make sure not to break any existing design choices.
  * Solve an issue: This is likely one of the two above, but it's about something that has already been publicly talked about. Which is a good thing! An unfortunate side effect is, that there may be a lot of explicitly stated and / or unwritten thoughts on the topic that you may need to incorporate. Also, some issues are arguable, some are in-progress and some are still too raw to be done. To guide you through this jungle, there's a [special label](https://github.com/AdamsLair/duality/labels/Up%20For%20Grabs) you can look out for. Issues flagged with this are most likely to benefit from external contributions.
  * Refactoring: Please don't, unless you've talked to Adam.
3. **Branch** your fork for this exact task, and nothing else.
4. **Implement** it and test it thoroughly.
  * Read this: [10 Tips For Better Pull Requests](http://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/)
  * If you're working on user interfaces, do usability testing.
  * Make sure it doesn't crash and always behaves non-destructive.
  * Make sure to obey the coding standards that are present throughout Duality. Just look through some code files and make sure your code looks exactly like everything else. You'll get a feel for it.
  * In case of new features, check how you addition feels in the overall framework, both regarding code design and usability. If it feels "tacked on", something is wrong.
  * Always use Tabs for indentation, but Spaces for alignment.
  * When in doubt, ask.
5. **Submit** a Pull Request and hope for the best.

Fork, Evaluate, Branch, Implement, Submit. Though, when doing this the second time, you already _have_ a fork and just need a new branch. 

All in all, developing the Duality Core may be the hardest part, mainly because it is the most integral one. It will require a lot of effort to get into how everything works, but that doesn't mean you can't start small. Pick something really easy at first, ideally a bugfix or micro-feature. You'll learn, and you're going to need practice and acquire some hands-on experience with Duality before moving on to something bigger. It's also a good idea to build some games with Duality first, in order to understand the workflow that comes with it, and the challenges that arise from it. 

Ah, and one more thing: It really can't hurt to ask whether work on a certain feature or issue is actually needed or desirable right now, before starting to work on it. Doesn't take long and might save everyone a lot of time :)
