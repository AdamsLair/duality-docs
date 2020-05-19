---
title: "Maintainer Guidelines"
category: "development"
displayOrder: -10
version: "v4"
---

This page outlines some of the guidelines we follow when developing Duality itself. Please take some time to read them before submitting a Pull Request - while they may not be equal to everyone's personal style, they do act as a common baseline that serves to make the codebase more readable and maintainable for everyone.

# General

There are three golden rules for applying the guidelines described here:

> **First:** Local guidelines override global guidelines.

Duality consists not only of one main project, but also uses several dependencies, some of which are forked third party projects. As such, they are primarily maintained by different people and inherit an entirely different set of rules. The guidelines described here **only apply unless overruled** by guidelines of a smaller scope, such as per-project guidelines in a dependency.

> **Second:** The guidelines are applied incrementally.

The guidelines found here describe a shared state that we work towards without ever fully reaching it. They are applied while working on different things, not as a task on their own. Even though older parts of the codebase may not implement all of these guidelines, it would be destructive to perform a "find replace" style change on them, causing merge conflicts in every custom fork of the project that happened to make changes there as well. Instead, we favor an **incremental approach**, where the current guidelines are applied whenever working on each part of the project, not in a single, big operation.

> **Third:** Everything is relative.

Guidelines only take you so far. There will always be cases that are the exception to the rule, and that's fine. These guidelines aim to provide a reasonable baseline that the project as a whole follows, but they do not describe every possible case. Deviate, but deviate within reason.

# Version Control

This section contains notes regarding the usage of version control in Duality.

## Commits

- The contents of a commit should be **completely intentional**. Any changes without distinguishable relation to the commits purpose are noise to be avoided.
- Be conscious about each file you commit. **Avoid "stage all"** / "commit all" operations.
- Be conscious about every line you commit. Avoid staging the whole file when the relevant change is just a few lines.
- Every commit should be a **self-contained unit** that aggregates related changes.
- Every commit should leave the project in a **working and useful state**.

## Commit Messages

- Commit messages are processed by automated CI tools in order to generate changelogs for package releases. The **message structure** of a commit should be as follows:

  ```
  One-Line Summary Describing the Purpose of the Commit
  
  #ADD: Addition of a new feature, API or behaviour.
  #REMOVE: Removal of a feature, API or behaviour.
  #FIX: Bugfixes of any kind, provided they do not fall into the above cases.
  #CHANGE: Changes in API or behaviour, as well as changes that do not fit any other category.
  ```

- The commit message should describe the **purpose and background of each change**, not mirror what's already in the git diff.

## Feature Branches

- Feature branches follow the naming scheme `feature/your-branch`.
- When working on a **feature that is too big** to fit into a small set of self-contained commits, a feature branch should be used instead. This allows to be less strict in commit contents, accounting for necessary work-in-progress states that are not fully functional yet.
- **Pull Requests should always** be based on feature branches. That way, a forked `master` branch remains operational without conflicts while the Pull Request is pending, adjusted, or squash-merged.
- When reviewing a feature branch before merge, its noise level should be considered.
  - **Low noise branches** consist mainly of self-contained commits that always leave a working and useful state. They are straight-forward and to the point. Branches like this can be merged in a regular merge commit, preserving their history as a useful part of the `master` branch.
  - **High noise branches** contain commits that temporarily break parts of the system, undo previous commits or revisit previous changes. They can be thought of as a search process that eventually arrives at a good solution. Branches like this should always be **squash-merged into a single commit** to keep the noise down on the `master` branch.

# Source Code

This section contains both style and semantic guidelines on developing the Duality codebase.

## Coding Guidelines

- Compiler **warnings should be fixed** as if they were errors. Aim for a zero-warning policy.

### Documentation

- Use **C# XML comments** for documenting API, not regular comments.
- All **public and protected API** needs to be documented for users of the Duality engine. Focus on describing usage and high level concepts while avoiding implementation details that aren't required to explain the former.
- Most **private API** should be documented for maintainers of the Duality engine. Describe usage and concepts, but also mention implementation details if they can be helpful for understanding how to make changes to each type.

### Design

- All members are **private by default**. If something doesn't need to be accessed, don't allow it.
- All properties are **getter-only by default**. Unless there's a need to write a value, it shouldn't be possible.
- All types are **transparent by default**. The documented "high level" behavior of a type should be enough to use it successfully. No knowledge of an objects internals should be required in order to use it according to its purpose, and there should be no way to enter an invalid internal state where the documented rules break down.
- A general exception to the above are data oriented structs such as colors, vectors and similar. For performance and usability reasons, they should favor public fields and allow direct access to their data.

## Style Guidelines

For using some of the style guidelines, we recommend enabling the display of whitespace characters in your IDE, as well as outlining / folding functionality that allows to collapse member definitions within classes.

### Whitespace

- Use **tabs for indentation**, but **spaces for alignment**.
- Be aware that **whitespace carries semantic value** and use it consciously and predictably, so it can act as a reliable cue when reading code.
- Every newline should have a specific reason to exist.
- The number of consecutive newlines is generally one, and should never exceed two.
- Do not use newlines to separate individual member definitions.
- Use newlines to separate semantically similar groups of member definitions.
- Use newlines to separate control flow into blocks that **form semantic units**.

### Comments

- A comment should **provide high level information and context**, not repeat what the code already says.
- In general, aim for **one comment per semantic block** of code. Besides helping other developers to understand the code, it also encourages a clear structure by organizing code in semantic blocks in the first place. Example:

```csharp
private void FooMethod()
{
	// A short description of what happens here
	int bar = this.DoSomething();
	string fooBar = this.DoSomethingElse(bar, 42);
	fooBar += "_SomeSuffix";

	// Another short description of the second block
	this.DoSomethingMore(fooBar);
	return fooBar;
}
```

- Don't use comments to stash unused code indefinitely, remove it instead.
- Temporarily commented out code should be tagged with a timestamp and remark describing the reason for its removal, so it can be safely cleaned up in the future.

### Casing

- Use **PascalCasing** for types, methods, properties, events and public fields.
- Use **camelCasing** for private fields, method arguments and local variables.
- Prefer full names over abbreviations, unless practicability dictates otherwise.

### Type Definitions

- Prefer one **type definition per file**. Do not group multiple types in the same file.
- File names should equal the name of the type they define.
- Avoid nested type definitions where possible.
- Accessibility modifiers (`private`, `protected`, `public`, `internal`) should be written out explicitly.