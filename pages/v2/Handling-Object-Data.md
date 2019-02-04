---
title: "Handling Object Data"
category: "advanced"
displayOrder: 0
version: "v2"
---

This article will provide a quick overview on the different ways in which object data interacts with Duality.

# Introduction

When defining a new custom object type such as Components or Resources, chances are that they will contain data, like health points, score values, follow targets or something similar. As part of your editor workflow and when running your game, Duality will need to interact with this data in three different ways - and while you usually don't need to worry about that, it can sometimes be useful to know what happens behind the scenes. The following sections will detail how exactly Duality will approach your objects, and what you can do to influence this.

## Basics: Field and Properties

There are two different language constructs in C# that are closely related to data storage: Fields and properties. They may look similar at first, but they serve very different purposes. In a custom Component or class, you can define them like this:

```csharp
public class Foo : Component
{
    // Field
    private float health = 100.0f;
    
    // Property
    public float Health
    {
        get { return this.health; }
        set { this.health = value; }
    }
}
```

**A field defines a slot** in your object that can hold a value. It's a variable that belongs to an object and it's purpose is to store data, like the `health` field storing a floating point number that represents how well the object is. This is pure data that can be read and written without causing much of a stir. Usually, fields are also declared as `private`, so no other object is allowed to access it - it's none of anyone's business, except for the object it belongs to.

```csharp
private float health = 100.0f;
```

On the other hand, **a property defines an access point** to retrieve or modify a value. In most cases, a property will act a `public` surface for an object's private data, gracefully allowing the object to react to changes from the outside, or transform its internal data before handing it to anyone else.

```csharp
public float Health
{
    get { return this.health; }
    set { this.health = value; }
}
```

Hiding data in private fields while providing public access points also allows you to define read-only variables (by omitting the setter) and helps you to think about what information an object really needs to expose, and which of it is really just an implementation detail.

# Inspection

The Object Inspector communicates with your objects similar to how your objects communicate with each other: Using the public API surface that you define. When displaying an object, it iterates over all **public properties** and provides a specialized editor for each. If one of those properties has no setter, it will be displayed read-only.

## Member Attributes

To change this behavior on a member-by-member basis, you can decorate them with `EditorHintX` attributes which will be evaluated by the Object Inspector when it decides how to display your object:

- `[EditorHintFlags(flags)]` allows you to specify any combination of the following:
  - `Invisible` will hide a member from the inspector when it would be visible otherwise. Hint: You can also use this with Component and Resource class definitions to make the editor skip them in Create menus.
  - `Visible` will show an otherwise hidden member in the inspector, such as properties declared as `private`.
  - `ReadOnly` will make the inspector display a member as read-only even though it might actually be writable.
  - `ForceWriteback` is a bit of a specialized matter: When you define a property of a reference type, this flag will make sure that modifying the returned child object in the inspector will trigger re-assigning it using your setter method. This allows you to react to changes that would otherwise go unnoticed.
  - `AffectsOthers` is a usability hint to the inspector that, after changing this value, it should update its display of all other properties of the same object as well, because they might have changed.
- `[EditorHintRange(min, max)]` allows you to set minimum and maximum values for your property. An optional extended version distinguishes between reasonable minmax (slider input) and limiting minmax (text field input). This only affects how the editor will display things. You can still assign whatever you like in code.
- `[EditorHintIncrement(value)]` allows you to adjust the size of a single value increment in a number editor and its slider.
- `[EditorHintDecimalPlaces(places)]` allows you to specify how many decimal places of a number are displayed in its editor. Users can still enter more accurate numbers manually.

## Custom Editors

To take all matters into your own hands, you can also go the extra mile and implement a custom editor for any object type, or even groups of object types that share a certain interface. Property editors are defined in editor plugins, which is a topic on its own that shouldn't be opened up in detail at this point - however, you can find some samples [here](/AdamsLair/duality/tree/master/Source/Editor/DualityEditor/Controls/PropertyEditors) and [here](/AdamsLair/duality/tree/master/Source/Plugins/EditorBase/PropertyEditors).

# Serialization

The term serialization refers to the process of transforming a complex object graph into a plain and simple stream of data, into a "series of bytes", that can then be stored inside a file or in memory. It is used when loading or saving Resources and other application data, when reading or writing a persistent representation of game data. It does _not_ play a role during object inspection.

When serializing an object, Duality will not use the public API surface that is defined by its properties, but **iterate directly over its fields**, whether they are private or not. The goal is not to store the visible subset of data that the object chose to make publicly available, but to make its entire internal state persistent. By default, all fields are serialized, but you can decorate any field (or type!) with a `[DontSerialize]` attribute to skip it.

If you want to take full control, you can also implement the `ISerializeExplicit` interface in any type and do it manually, or implement a `SerializeSurrogate<T>` to represent a type that you have no control over, like types defined in a library you didn't write.

## Supported Data

It is usually safe to assume that Duality can serialize it, even if it is not based on a Duality type. The following list should give you an idea of what it can do:

- Primitive data types
- Enums
- Structs
- Class instances and references to them (while maintaining identity)
- Delegates
- Type or MemberInfo data
- Arrays, lists, dictionaries and other collections
- Boxed value types
- References that hide the true type of an object, so whatever is behind that `private object foo;` field, it will be serialized as expected. Works with both inheritance and interface implementations.
- Nested structures including any of the above in any configuration, such as a `Dictionary<Type,List<CustomClass[][]>>` even if `CustomClass` contains references to more objects.
- It doesn't matter in which Assembly a type is defined

## Unsupported Data

The following is a list of exceptions from the above list of supported data:

- `IntPtr` or `UIntPtr` data
- Native pointers
- Multi-dimensional arrays (don't use `[5, 5]`, use `[5][5]` instead)
- Objects that store runtime-only data, or do not expect to be serialized. Mind what you serialize, and add a `[DontSerialize]` attribute to the fields that you want to omit.

# Cloning

Copying and cloning objects is handled by a different subsystem of Duality and generally is a much more complex operation. When serializing something, it is the assumption that the object (graph) we're dealing with is already a self-contained unit, like a Pixmap, a Prefab or a Scene - so all referenced objects can simply be dragged into the serialization process and saved along with the rest. When cloning an object, we do not have this luxury and thus the question of ownership becomes a central topic: Does `Foo` _own_ `Bar`, or is it merely _referencing_ it? When cloning `Foo`, will the clone still reference the same `Bar` as the original `Foo`, or one that has been cloned as well? To complicate things even more, copy operations may also target an object graph that partially exists already, requiring a degree of object re-use. It's not the goal of this article to go in too much detail on this, but it is useful to know that there is a certain complexity in the depths of this operation.

Typically, cloning is used when duplicating an object in the editor, when using the `DeepClone()` or `DeepCopyTo()` extension methods or when applying or instantiating a Prefab. It also plays an extensive role in the Undo / Redo system of the editor.

In 99% of all cases, you don't have to worry at all about your objects cloning behavior. By default, **all fields that are serialized will also be cloned**, so `[DontSerialize]` will also have an effect on cloning. It is also assumed that all objects that do not derive from `Component`, `GameObject` or `Resource` are owned and not referenced, thus they will be deep-cloned along with the object that holds their reference. You can alter cloning behavior on a per-field and per-type basis using the `[CloneField]` and `[CloneBehavior]` attributes.

If you want to take full control, you can also implement the `ICloneExplicit` interface in any type and do it manually (see also the `[ManuallyCloned]` attribute), or implement a `CloneSurrogate<T>` to represent a type that you have no control over, like types defined in a library you didn't write.

