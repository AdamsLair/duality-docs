---
title: "Package Management"
category: "advanced"
displayOrder: 0
version: "v4"
---

If you’re a C# developer, there’s a good chance that you already know what [NuGet](https://www.nuget.org/) is, or at least have overheard someone talking about it. The main idea is to stop delivering precompiled dependencies along with source code, and instead provide a central repository where all these binary packages are stored. Whenever someone needs one of them, it can be downloaded automatically, and whenever a new version is available, upgrading is only a mouse click away. Package Management is just incredibly convenient – and by now, Duality is able to make use of it. Let’s take a quick look at how it works.

# Package Management Basics

This chapter describes some concepts behind Package Management in Duality, and how to use it in your development cycle.

## What is a Package?

Duality has been built with a modular architecture in mind, which means it can be split up into a lot of distinct parts - like the Scene View, Help Advisor, Steering, Dynamic Lighting and many more. Because most of these parts don't interact directly with each other, it's easy to add, remove or switch each part individually, without breaking the whole system. This is modular design. Now, in order to make those modules more conveniently accessible, all the files of a single module can be packed into a single one and annotated with a description of what's inside. This is what's called a Package.

## How can I use Packages?

The main feat of Package Management is added convenience. 

As of v4 the integrated package manager in duality was removed and now uses [Package References](https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files), the default way of managing packages in .NET. This means you are free to choose your tool of choice to manage your packages: 
- [dotnet tooling](https://docs.microsoft.com/en-us/nuget/reference/dotnet-commands)
- [integrated package manager in visual studio](https://docs.microsoft.com/en-us/nuget/consume-packages/install-use-packages-visual-studio)
- just manually edit the csproj file.

## Updating Packages

Unlike installing or uninstalling Packages, updating a Package always involves the danger to introduce breaking changes. As a general guideline, you can take a close look at the version number of the update. Version numbers always have the following format:

```
Major.Minor.Patch
```

When assuming that the author of the Package obeys semantic versioning rules or Duality versioning guidelines, Patches never introduce breaking changes, Minors usually don't and Majors can pretty much do everything. They may still play nice and be completely backwards compatible, but there's no actual restriction for them to be.

Depending on the kind of changes that have been introduced with a Package update, you may need to recompile and potentially fix your custom game plugin, and the safest way to do so is before touching any game content.

## Packages? Online? Can I opt-out?

Yes, you are free to only use file and project references in your projects. Duality does not care and is fully functional without any kind of Package Management as long as the files end up in the same location. You can also specify a different package repository using a [Nuget.config](https://docs.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior) file, this can even be a local folder.

# Publishing Packages

Another cool thing about Package Management is, that everyone is free to introduce new Packages into the mix - and they will be available to all Duality users just like the official ones. Here's a quick overview on how to get started:

1. Create a new Duality plugin. Bonus points for hosting it on GitHub. If you need examples on how a Duality plugin project looks like, you may want to take a look at official [Core](https://github.com/AdamsLair/duality/tree/master/Source/Plugins) and [Editor](https://github.com/AdamsLair/duality/tree/master/Source/Plugins/EditorModules) plugins.
2. Develop a first stable or prototype version that is somewhat ready to be used.
3. Get a [NuGet account](https://www.nuget.org/) and read about [how to publish Nuget Packages](https://docs.microsoft.com/en-us/nuget/nuget-org/publish-a-package).
4. In most cases using the project file itself should suffice to create a nuget package with the [dotnet cli](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package-dotnet-cli). Some things to note:
  - Your Package `id` should be structured like this: `YourName.Duality.Plugins.PluginName` for Core plugins or `YourName.Duality.Editor.Plugins.PluginName` for Editor plugins.
  - In order for your Package to be identified as Duality Package, `PackageTags` should contain both `Duality` and `Plugin`. it is also a useful guideline to include `Core`, `Editor` or `Sample`, depending on what your Package contains. Do not combine them, but instead publish distinct packages for Core, Editor and Samples.
  - You need to explicitly specify the version of your Package. It is strongly recommended to use the above guidelines on Major, Minor and Patch numbers. Don't specify a Build number.
  - As far as dependencies go, Core plugins should at least depend on `AdamsLair.Duality` and Editor plugins should at least depend on `AdamsLair.Duality.Editor`. Feel free to add more dependencies to other (Duality and non-Duality) Packages when required.
  - **Do not** explicitly refer to any of the Assemblies that are already included in Duality itself, e.g. **do not** refer to `AdamsLair.OpenTK`, `AdamsLair.WinForms` or similar. Those are to be considered part of the overall environment.
  - A full list of metadata properties van be found [here](https://docs.microsoft.com/en-us/dotnet/core/tools/csproj#nuget-metadata-properties)
5. Run `dotnet pack` on your project file to generate the nuget package.
6. Upload your nuget package to nuget
  - **do not** use the online editor of the NuGet Gallery to edit any of the properties you've set in the `.nuspec` file, as this [will apparently introduce inconsistencies](https://forum.duality2d.net/viewtopic.php?p=5003#p5003) between the gallery entry and the package.

Alternatively you can use the [nuget.exe cli](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package) to pack a `.nuspec file`. Some examples can be found [here](https://github.com/AdamsLair/duality/tree/master/Build/NuGetPackageSpecs). Its recommended however to use the `dotnet cli`.