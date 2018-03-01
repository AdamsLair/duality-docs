---
title: "Package Management"
category: "advanced"
displayOrder: 0
version: "v2"
---

If you’re a C# developer, there’s a good chance that you already know what [NuGet](https://www.nuget.org/) is, or at least have overheard someone talking about it. The main idea is to stop delivering precompiled dependencies along with source code, and instead provide a central repository where all these binary packages are stored. Whenever someone needs one of them, it can be downloaded automatically, and whenever a new version is available, upgrading is only a mouse click away. Package Management is just incredibly convenient – and by now, Duality is able to make use of it. Let’s take a quick look at how it works.

# Package Management Basics

This chapter describes some concepts behind Package Management in Duality, and how to use it in your development cycle.

## What is a Package?

Duality has been built with a modular architecture in mind, which means it can be split up into a lot of distinct parts - like the Scene View, Help Advisor, Steering, Dynamic Lighting and many more. Because most of these parts don't interact directly with each other, it's easy to add, remove or switch each part individually, without breaking the whole system. This is modular design. Now, in order to make those modules more conveniently accessible, all the files of a single module can be packed into a single one and annotated with a description of what's inside. This is what's called a Package.

## What can it do for me?

The main feat of Package Management is added convenience. Because all of the Packages are stored in a central online repository to which Duality can connect, you don't even need to leave the editor in order to install or uninstall new Packages. Updating is just a mouse click away and is similarly automated. 

To access Package Management functionality, open the `File` menu and click `Manage Packages...`. A user dialog will appear to guide you through everything else.

## Updating Packages

Unlike installing or uninstalling Packages, updating a Package always involves the danger to introduce breaking changes. As a general guideline, you can take a close look at the version number of the update. Version numbers always have the following format:

```
Major.Minor.Patch
```

When assuming that the author of the Package obeys semantic versioning rules or Duality versioning guidelines, Patches never introduce breaking changes, Minors usually don't and Majors can pretty much do everything. They may still play nice and be completely backwards compatible, but there's no actual restriction for them to be.

Depending on the kind of changes that have been introduced with a Package update, you may need to recompile and potentially fix your custom game plugin, and the safest way to do so is before touching any game content.

## Packages? Online? Can I opt-out?

Yes. Duality is fully functional without any kind of Package Management and doesn't require it in any way. Removing it entirely from Duality requires a few steps:

1. To make the Package dialog go away permanently, open the Package dialog and uninstall the `Package Manager Frontend` Package. (Yes, it can actually uninstall itself.) Restart the editor to apply the uninstall progress.
2. Close the editor again and delete `PackageConfig.xml` from your Duality folder.
3. If you want to save some disk space, delete `Source/packages` as well. 

If you're proficient with NuGet, you can also use a custom Package repository instead of the default one. Open 
`PackageConfig.xml` and replace the appropriate URL with the one you want to use. You can also enter a relative folder path on your local disk.

# Publishing Packages

Another cool thing about Package Management is, that everyone is free to introduce new Packages into the mix - and they will be available to all Duality users just like the official ones. Here's a quick overview on how to get started:

1. Create a new Duality plugin. Bonus points for hosting it on GitHub. If you need examples on how a Duality plugin project looks like, you may want to take a look at official [Core](https://github.com/AdamsLair/duality/tree/master/Source/Plugins) and [Editor](https://github.com/AdamsLair/duality/tree/master/Source/Plugins/EditorModules) plugins.
2. Develop a first stable or prototype version that is somewhat ready to be used.
3. Get a [NuGet account](https://www.nuget.org/) and read about [how to publish Nuget Packages](http://docs.nuget.org/docs/creating-packages/creating-and-publishing-a-package).
4. Create a `.nuspec` file for your new Duality Package. There is also an [official specification](http://docs.nuget.org/docs/reference/nuspec-reference) of these, in case you need it.
  - Your Package `id` should be structured like this: `YourName.Duality.Plugins.PluginName` for Core plugins or `YourName.Duality.Editor.Plugins.PluginName` for Editor plugins.
  - In order for your Package to be identified as Duality Package, `tags` should contain both `Duality` and `Plugin`. it is also a useful guideline to include `Core`, `Editor` or `Sample`, depending on what your Package contains. Do not combine them, but instead publish distinct packages for Core, Editor and Samples.
  - You need to explicitly specify the version of your Package. It is strongly recommended to use the above guidelines on Major, Minor and Patch numbers. Don't specify a Build number.
  - The Package `title` and `summary` is what will be displayed to the user in the list of available Packages.
  - The `description` is what they will see after selecting it. it may be much larger and more thorough than the summary.
  - `projectUrl` and `iconUrl` will be used accordingly. If you hosted your project on GitHub, I'd recommend to upload the icon (32x32 .png) there as well and specify the direct / raw image link as icon url and the GitHub project page as project url.
  - In the list of `files`, you should include every file that belongs to your plugin.
    - Binary files go to the "lib" `target`. **Do not** include files that are actually part of a dependency.
    - Content / Data / Resource files go to the "content" `target`, or a subdirectory of it.
  - As far as dependencies go, Core plugins should at least depend on `AdamsLair.Duality` and Editor plugins should at least depend on `AdamsLair.Duality.Editor`. Feel free to add more dependencies to other (Duality and non-Duality) Packages when required.
    - **Do not** explicitly refer to any of the Assemblies that are already included in Duality itself, e.g. **do not** refer to `AdamsLair.OpenTK`, `AdamsLair.WinForms` or similar. Those are to be considered part of the overall environment.
  - You can find plenty of examples [here](https://github.com/AdamsLair/duality/tree/master/Build/NuGetPackageSpecs).
  - Also, **do not** use the online editor of the NuGet Gallery to edit any of the properties you've set in the `.nuspec` file, as this [will apparently introduce inconsistencies](http://forum.adamslair.net/viewtopic.php?p=5003#p5003) between the gallery entry and the package.
5. Let NuGet build the package and push it to the repository. It will be available to everyone in a few minutes.