---
title: "Custom Editor Actions"
category: "advanced"
displayOrder: 0
version: "v4"
---

This page describes how to extend the editor with custom Editor Actions.

# What is an Editor Action?

An Editor Action is a generic action that can be taken within the editor. These most often correspond to context menu actions but can also be invoked upon opening a resource or preparing an object for editing. When defining an Editor Action you implement the logic that determines whether or not your action can be performed in a given context. Simply defining the Editor Action is enough to integrate them with Duality. The editor will automatically populate context menus and execute your defined actions as needed.

# Defining an Editor Action

All Editor Actions extend from `IEditorAction`. By implementing this interface inside an editor plugin, your `IEditorAction` will automatically be exposed to the editor. Although implementing `IEditorAction` is one way to create Editor Actions, it is generally preferable to extend from the `EditorAction<T>` or `EditorSingleAction<T>` classes as doing so will reduce the amount of boilerplate code you have to write. Additionally, `EditorAction<T>` and `EditorSingleAction<T>` allow you to write strongly typed Editor Actions whereas `IEditorAction` deals with `object` typed data.

## EditorAction\<T\>

Extending `EditorAction<T>` will provide a reasonable default implementation for the `IEditorAction` interface and will allow you to perform the action on objects of type `T`.

As an example, here is an Editor Action that adds a context menu entry to `Pixmap` resources that opens a custom window when performed:

```csharp
public class OpenMyWindow : EditorAction<Pixmap>
{
	public override string Name
	{
		get { return "Open My Window"; }
	}

	public override void Perform(IEnumerable<Pixmap> pixmaps)
	{
		MyWindow window = new MyWindow();
		window.SetPixmaps(pixmaps);
		window.Show();
	}
}
```

## EditorSingleAction\<T\>

`EditorSingleAction<T>` is essentially the same class as `EditorAction<T>` but allows you to write Editor Actions that will be executed on a single object at a time. When multiple objects are selected, your `Perform` implementation will be called for each object, one at a time.

For example, the following is the Editor Action used to convert `Pixmaps` to `Textures` (unimportant details removed):

```csharp
public class PixmapToTexture : EditorSingleAction<Pixmap>
{
	public override string Name
	{
		get { return "Create Texture"; }
	}

	public override void Perform(Pixmap obj)
	{
		// Get a path for the new texture resource
		string texPath = PathHelper.GetFreePath(obj.FullName, Resource.GetFileExtByType<Texture>());
		Texture tex = new Texture(obj);
		tex.Save(texPath);
	}
}
```

If this Editor Action had instead extended from `EditorAction<Pixmap>` its `Perform` implementation would take an `IEnumerable<Pixmap>`. That wouldn't be a major problem, but using `EditorSingleAction<T>` requires less code in this case and is a bit more convenient.

# Action Contexts

The examples shown so far add options to the context menu of a type of object in the editor. There are other contexts in which editor actions may execute. Commonly used contexts are as follows:
- DualityEditorApp.ActionContextContextMenu
  - Used when populating context menus
  - This is the default context for `EditorAction<T>` and `EditorSingleAction<T>`
- DualityEditorApp.ActionContextOpenRes
  - Used when opening a resource in the editor
- DualityEditorApp.ActionContextSetupObjectForEditing
  - Used when initializing an object for editing, such as when it is newly created in the editor

You can react to these different contexts by overriding the `MatchesContext` method. For example, the following override would mark this Editor Action as suitable when opening a resource of type `MyResourceType`:

```csharp
public class MyEditorAction : EditorAction<MyResourceType>
{
	/* Rest of implementation skipped */
	public override bool MatchesContext(string context)
	{
		return context == DualityEditorApp.ActionContextOpenRes;
	}
}
```

It is also possible to define your own [actions contexts](#custom-actions)

# Using Editor Actions

If all you want to do is add a context menu to a given object type or process a resource when it is open or loaded, simply defining the Editor Action in an Editor Plugin is enough - Duality will find it through Reflection. If, however, you need to get the list of available editor actions yourself or need to add custom action contexts, you need additional information.

## Retrieving Editor Actions

Editor actions matching certain parameters can be retrived by calling `DualityEditorApp.GetEditorActions`. For example, the following call gets all editor actions that can operate on objects of type `GameObject`, can be performed on the given set of objects, and can be performed in the context of opening a context menu:

```csharp
List<GameObject> targetObjects = /* get the objects to perform the actions on*/;
IEnumerable<IEditorAction> actions = DualityEditorApp.GetEditorActions(
	typeof(GameObject), 
	targetObjects, 
	DualityEditorApp.ActionContextContextMenu);
```

## <a name="custom-actions"></a>Custom Action Contexts

If you are extending the editor more extensively and perhaps adding your own windows or something similar, you may need to add a new context string to limit your `GetEditorActions` call to the right actions. This is not difficult at all. Simply choose a string to use and pass that to `GetEditorActions`: 

```csharp
DualityEditorApp.GetEditorActions(typeof(TargetType), targetObjects, "NewContext");
```

Then, modify your Editor Actions so `MatchesContext` returns true when given that context:

```csharp
public class MyEditorAction : EditorAction<TargetType>
{
	/* Rest of implementation skipped */
	public override bool MatchesContext(string context)
	{
		return context == "NewContext";
	}
}
```

It is common practice to store action context strings as constants inside of your `EditorPlugin` class, but you do not need to.
