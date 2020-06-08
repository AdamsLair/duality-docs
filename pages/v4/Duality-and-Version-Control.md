---
title: "Version Control"
category: "advanced"
displayOrder: 0
version: "v4"
---

This article will outline how to use version control systems such as git or Subversion with a Duality project. It does not cover how to use any specific version control system in general, as there is extensive documentation for each of them out there on the web.

# Introduction

Let's start this article with some general words of advice, in case you are not familiar with a version control system yet, or have not yet applied one to a game development project.

## What is Version Control?

Think of it as a sophisticated backup and collaboration strategy. When working on a project, you will gradually introduce changes. Most of the time,  these changes will be done in several work sessions, and after a few sessions, you might decide to make a backup to have a safe haven to return to, should any of your future changes screw something up.

Version control systems are a way to automate your workflow and organize all versions of all the files in a certain place, so you don't have to bother. On top of that, they are also able to do things that would require a tremendous organization effort otherwise, such as merging two different versions of your project after you and a colleague have worked on it independently for a while.

## Why Should You Use Version Control?

In software and also game development, relying on some kind of a version control system is usually considered a no-brainer. Here's why:

- You gain the **ability to undo any change** indefinitely, right back to the beginning of your project.
- Your project is **stored in more than one place**, so you won't lose it when hardware fails or something gets deleted that shouldn't be.
- It makes it far **easier to collaborate** with others and keep everyone in sync, as well as branch off for a while and re-join later.
- It prompts you to be more **conscious about recent changes** after each work session, because you need to acknowledge them as part of your workflow.

## Which System Should You Use?

There are lots of version control systems out there and arguably, there is no "right" choice. Research it! You will find lots of comparisons out there, but there are some general things to consider as well. Let's pick **git** and **Subversion** as two different examples for version control systems.

### Subversion

Subversion, also known as SVN, is a centralized version control system: There is one repository per project, which is considered the ground truth - all other versions are just checkouts of it, momentary snapshots if you will. A popular UI-based Windows client for it is [Tortoise SVN](https://tortoisesvn.net/).

It's easy to learn, especially if you haven't used version control before, and it's built on a very simple concept: There's the repository and there's the checkout version. You work in the checkout version, commit your changes back to the repository once in a while and update to retrieve the latest changes from anyone else. That's about it.

The fact that Subversion is a centralized system is both an advantage and a disadvantage: You can't use version control when the central repository in unreachable or otherwise unavailable, because that repository is the ground truth everyone is referring to. On the other hand, since there _is_ a central repository, you don't need to store the entire history of your project locally - it's all stored safely on the server. This is especially useful when working with large game assets, because you only need to download the latest version, not all of them.

### git

git, unlike Subversion, is a decentralized version control system: Every repository is a full clone of the original. A popular UI-based Windows client for it is [SourceTree](https://www.sourcetreeapp.com/).

Git is [a little more complex](https://xkcd.com/1597/) than Subversion, but, once the basic workflow has settled in, not a tiny bit less productive. Its decentralized approach makes juggling five slightly different branches of the project feel more natural and many consider git to be superior to (for example) Subversion - although one [could argue](http://stackoverflow.com/a/875/2015377) it's mostly a matter of preference.

In practice, being decentralized means that you can work as usual regardless of whether or not you can reach the server, with all the version control features you're used to - just locally. When you can reach the server again, or decide to make your recent local changes public, they can be pushed back easily.

However, being decentralized also means that every clone of the repository contains the entire history of the project. If you commit a 2GB file in the beginning only to delete it soon after, every future clone will _still_ have to download that 2GB file. A similar problem awaits you when iteratively editing lots of semi-large asset files (images, audio) because every version in your projects history will always be part of the download. It's usually manageable for a reasonable amount of small to medium files, but large, asset-heavy projects often require a plan B.

### Others

Of course there are more than those two, and the choice is yours. The most important thing is that you _use_ version control. _Which_ one is not the highest priority to begin with, but once you are familiar with the details you will find what suits your use cases best.

# Version Control in Duality

If you're an experienced version control user, you know that you sometimes need to be careful which files to add to the system, and which ones to keep out of it. In fact, adding _all_ files of a Duality project to version control would be an incredibly wasteful thing to do. It's always good to keep your repository nice and clean, with only the essential files persistently versioned, while keeping out redundancy and temp noise. In v4 duality made this easier to achieve because all binaries are now deployed from the project files in your solution so its no longer needed to put these under source control.

## Duality as a Self-Installing Application

When you initially downloaded and installed Duality, you may have noticed that there was no dedicated installer software, only a solution file with some projects. Building it for the first time will download all the required packages and copies them into place. After this you can run Duality.

This is not a one-off operation - if you delete all files except for the ones that came with the download, you can simply build your solutions again to restore these files. We can use this behavior to keep our version control repository clean of all binaries.

## The Initial Setup Commit

The duality project template already comes with a pre-configured .gitignore file for you so you don't have to do this yourself. However it is good to know why some files are ignored and others are not.

Let's go through the most important files and folders to ignore, and see _why_ they should be ignored:
- Build output files (`**/bin`, `**/obj`, `Plugins`, `*.dll` etc) are the output files/folders building your game so we don't need these. Ignore!
- `Backup` because that's what our version control system already does.
- `Temp` these contain only temporary files which are not needed to be under source control.
- `/.vs` this folder contains temporary files used by visual studio.

### Example Git Ignore File

If you're using **git**, here is a sample `.gitignore` file from the template for you to start with. 

```
# Build output files
**/bin
**/obj

# Duality specific files
/Temp
/Backup
/UserData.dat
/DesignTimeData.dat
*.pdb
/*.dll
/*.exe
/*.txt
/*.xml
/Plugins

# Visual studio specific files
/.vs
```