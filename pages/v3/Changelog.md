---
title: "Changelog"
category: "development"
displayOrder: 1000
version: "v3"
---

This page provides an overview of changes that were introduced in the major version change between Duality v2.x and Duality v3.0. Any further changes will be documented in the release notes of the individual packages, or on the changelog page of the next major version change.

* TOC
{:toc}

# Highlights

The following is a high level overview on the general nature of changes in this major version step. 

- **Rewrote the entire rendering pipeline**, allowing massive improvements in efficiency, ease of use and feature coverage while at the same time shifting towards more modern, shader-focused rendering techniques.
- **RenderSetup resources** allow replacing how Duality renders a frame, providing a simple entry point for pre- or post processing steps, as well as completely customized rendering behavior.
- **Self-contained scenes** allow using `Scene` resources as isolated simulation spaces independently of the active main scene.
- **Focus on performance** and a more data-oriented design of both rendering and update cycles. A worst-case rendering benchmark comparing v2 and v3 performance went down from 13 ms to 6 ms per frame, as well as 2000+ (gen0 to gen2) GC collections per minute to about 5 (gen0) collections.
- **Refactored core API**, replacing many "first iteration" designs with more streamlined ones. The most prominent example might be the deprecation of manual context checks in `ICmpInitializable` in favor of a simple `OnActivate` / `OnDeactivate` method pair, but many similar improvements were done as well.

For more detailed information, check out the "All Changes" section down below.

# All Changes

The following is a low level overview on all changes that were introduced between Duality v2.x and Duality v3.0, with the exception of some trivial changes such as renamed private fields, minor implementation improvements or similar.

## Framework Structure

- Extracted smooth animation shaders, techniques and code from the core and moved it into the new `SmoothAnimation` sample project.
- Integrated the previously external `FarseerDuality` physics library into the main project as `DualityPhysics` to ease future optimizations.
- Added a new `Benchmarks` sample project that provides a controlled environment for testing the impact and effects of various rendering techniques and optimizations.
- Added a new `CustomRenderingSetup` sample project that demonstrates how to use and extend the new `RenderSetup` Resource type to define how Duality renders a frame.

## Rendering

- Updated from OpenGL 2.1 to OpenGL 3.0 core profile, and removed all legacy OpenGL code.
- Updated the internal `OpenTK` dependency and fixed a context creation glitch that would create a legacy OpenGL context by accident and prevent tools like RenderDoc from interacting with the Duality rendering output.
- Added `RentMaterial` API to `IDrawDevice`, allowing renderers to use temporary materials without any per-frame allocations.
- Added new `CanvasState` API to promote efficient material renting and re-use with added convenience.
- Integrated `CanvasBuffer` into `Canvas`, which internally re-uses old vertex data to avoid allocations. The former `RequestVertexArray` functionality is now available through `RentVertices`.
- `Canvas` now requires enclosing all drawing commands to a specific device in a `Begin` / `End` block, allowing to share and re-use the same `Canvas` object between multiple renderers and across multiple drawcalls.
- Optimized push / pop of `Canvas` states to re-use the same instances internally, avoiding allocations.
- The `RenderOptions` class now allows to specify whether depth writing and depth testing are enabled, as well as a global shader parameter collection.
- Added a property to explicitly specify whether a `RenderTarget` has a depth buffer.
- The editor graphics context can now be created with a specific antialiasing quality that is independent from game settings.
- Moved all visual picking functionality from `Camera` to `PickingRenderSetup`, leaving only a thin API wrapper in place for convenience reasons.
- Added new `BatchInfo` and `Material` constructors to ease specifying color-only materials without a texture.

### Pipeline Rewrite

- Introduced the new `RenderSetup` Resource that defines how Duality renders a frame and provides a convenient way to take control of this, or replace the existing render pipeline with a custom one. Superseded `Camera` passes with `RenderStep` items in a `RenderSetup` and greatly simplified `Camera` implementation and API.
- Conceptually separated viewport and rendered image size, making it trivial to render the same view in lower or higher resolution, or adjust the target area it will be rendered to.
- Added a `ForcedRenderSize` option to appdata settings, allowing games to render with the same view regardless of window or screen size. Differences in aspect ratio can be accounted for either by letterboxing or by displaying additional content. User input is automatically re-mapped from actual window coordinates to forced view coordinates.
- Reworked `ICmpRenderer` to no longer expose logic-related methods for determining visibility, but instead provide a single `GetCullingInfo` method. The retrieved plain-old-data struct contains all culling relevant information, which is now cached and batch-processed by the scenes visibility strategy implementation.
- Separated culling updates from culling queries, improving performance by avoiding redundant work for rendering multiple viewports or passes.
- All vertex transformation now takes place on the GPU, and all renderers now specify vertices in world space, rather than pre-transformed intermediate view space. The `IDrawDevice` interface no longer has a `PreprocessCoords` methods, and there is no need for it anymore either.
- Reimplemented `DrawDevice` to streamline vertex processing and dynamic batching. Vertex storage and rendering are now treated separately and using specialized data structures, allowing more efficient batching, without the overhead of any per-frame allocations. Introduced the new `VertexBatch<T>` / `VertexBatchStore` helper classes for both internal usage and custom dynamic batching implementations.
- The `DrawBatch` class has been rewritten, is now public API, and can be managed and submitted manually when needed.
- Removed `IDrawBatch` in favor of direct `DrawBatch` usage, avoiding virtualization and easing compiler optimization. 
- Introduced the new `VertexBuffer` class that represents a GPU side vertex / index data combo and allows to manually setup, load and update them in regular core and game code as needed. For vertex-heavy and potentially static objects such as tilemaps, this is more efficient alternative to re-submitting vertex data every frame.
- Removed `RenderMode` and `RenderMatrix` enums, as they were superseded by explicit view / projection matrix handling. Introduced `ProjectionMode` enum to specify what kind of projection matrix is used by a drawing device.

### Shaders

- Updated default GLSL version from 1.2 to 1.3, and adjusted all builtin and sample shaders accordingly.
- All shader parameters are now specified explicitly in shader code. This includes all uniforms and vertex attributes, as there no longer are any builtin variables.
- Added `#pragma duality *` support to GLSL shaders, allowing to specify variable metadata such as descriptions, minmax or type information.
- Replaced hardcoded `VertexElementRole` assignments with a name-based `VertexElement`-to-attribute lookup. This makes it far easier to specify and use custom vertex elements in both C# and GLSL code.
- Added `ShaderParameterCollection` class for efficiently storing and comparing shader parameters.
- Reworked `BatchInfo` and `Material` data and API to make efficient use of `ShaderParameterCollection`. There are no longer special cases for main color or texture, as they are now regular shader parameters as any other.
- Merged `ShaderProgram` resources into `DrawTechnique` resources, which now directly specify a vertex and fragment shader.
- There is now an implicit global shader include that provides a base interface for common, Duality-specific functionality, such as `AlphaTest(alpha)`, `TransformVertexDefault(worldPos, depthOffset)` or `TransformWorldToView(worldPos)`.
- There are now rendering operation wide / "global" shader variables, which are both available via public API and used for builtin shader variables. Global shader variables are set per-shader, not per-batch.
- Shader compile and link messages are now translated into one Duality log message each, making it easier to debug issues with shader code.
- Added the new `ShaderSourceBuilder` utility class to pre-compile and merge multiple shader source files into one.

## Physics

- The internal physics world is now an explicitly managed part of a scene, and accessible via the `scene.Physics` property.
- Moved all physics queries from static `RigidBody` methods to `PhysicsWorld`.
- Removed obsolete `GearJoint` and `SliderJoint` due to unstable behavior.
- Unified most functionality of `ChainShapeInfo`, `LoopShapeInfo` and `PolygonShapeInfo` in their new common `VertexBasedShapeInfo` base class.

## GameObjects and Components

- Simplified `ICmpInitializable` to only require `OnActivate` and `OnDeactivate` methods, without any init context checks. The remaining init context equivalents have been extracted into `ICmpSerializeListener` and `ICmpAttachmentListener` interfaces that provide their own, specialized API. `InitContext` and `ShutdownContext` no longer exist.
- Reworked `GameObject` API and optimized its implementation, especially with regard to component retrieval.
- Renamed the `GameObject.ParentScene` property to `Scene`, and introduced a shortcut `Scene` property to `Component`.
- Introduced the `ICmpSpriteRenderer` interface that defines shared API for all renderers displaying a(n indexed) sprite.
- Replaced `AnimSpriteRenderer` with `SpriteAnimator`, which contains no rendering functionality, but can animate any `ICmpSpriteRenderer`.
- Added sprite / atlas indexing support to `SpriteRenderer`.
- Unified all renderer depth offset properties to use a Z axis float offset, rather than some abstract integer value.
- The `Transform` component no longer tracks or serializes velocity values, and it no longer implements `ICmpUpdatable`, boosting performance and reducing memory usage significantly. Add a `VelocityTracker` component to re-enable tracking on specific objects.
- Reworked `Transform` API to be more consistent and intuitive.
- Removed `DeriveAngle` (false) support from `Transform`.
- Removed obsolete API from `Component`.

## Resources

- Automatic slicing of images into animation frames has been moved completely out of `Pixmap` and into asset import and the new Pixmap Slicer UI.
- Reworked and partially re-implemented `Font` resources to fully embrace bitmap font features, including kerning pairs and extended glyph geometry information. Removed all dependencies to TTF rendering, which was moved into asset import. A lot of specialized code was moved into separate (public) classes, such as `FontData` or `FontKerningLookup`.
- Scenes can now be de/activated, updated and rendered explicitly, independent from which scene is `Current`. Reduced overall reliance on the static `Scene.Current` property and moved all rendering, physics and simulation code towards requiring an explicit `Scene` parameter.
- Added a new `ContentRef<T>` constructor that accepts a path string only.
- Renamed `ContentRef.MakeAvailable` to `EnsureLoaded` to reflect its purpose more closely.
- Cleared `ContentProvider` API from unused methods and moved some of them to the `Resource` class.
- Added the new `DefaultContent` helper class that now houses all related init code.
- Adjusted `Texture` resource API for convenience, and to match other changes in the project.

## Backend

- Changed API of `INativeAudioBuffer`, `INativeTexture`, `INativeRenderTarget` and `IGraphicsBackend` to support data buffer arguments of type `IntPtr`, and added generic extension methods to allow specifying array data of any kind. This decouples the declared data type from the actual data and allows to re-interpret plain-old-data struct or primitive arrays as needed.
- Introduced `INativeGraphicsBuffer` as a generic graphics data storage interface, allowing the core to explicitly manage vertex and index data in the same way that other graphics resources are managed.
- Renamed `IPluginLoader` to `IAssemblyLoader` and adjusted its default implementation accordingly.
- Adjusted `WindowOptions` data and API to match other changes in the project.
- Fixed a bug in the default OpenTK backend that would cause OpenTK to be initialized even when a different backend was chosen.

## Editor

- The `TransformPropertyEditor` in the inspector now defaults to show end edit local space values, with an option to switch to world space values.
- The `CamView` now allows to explicitly specify the game sandbox rendering resolution and re-maps user input to match.
- The `CamView` now allows to explicitly specify the `RenderSetup` to use when rendering regular editing views.
- Optimized `CamView` gizmo rendering to use `Canvas.ColorTint` instead of different materials to improve batching.
- Optimized `CamView` mouse picking to use half resolution scene renderings.
- The `FileEventManager` class has been re-implemented, tested and optimized with various previously unknown bugs fixed in the process. Its API was redesigned to emit events in batch processing mode, rather than one event at a time.
- The `ProjectView` now properly handles error cases where moving a file or folder fails because of missing write access.
- Optimized `SceneView` object insertion to reduce delay when creating a lot of objects in bulk.
- File change events on `Resource` files are now automatically translated into `PropertyChanged` events for their loaded instances.
- Text descriptions for `EditorAction<T>` are no longer part of their API, but retrieved from XML documentation.
- Removed buggy JIT debugging capability detection and instead enabling the editor debug button in any case.
- Removed obsolete `DataObject` extensions API.

## Logging

- Cleaned up logging API and split functionality into `Log`, (static) `Logs` and (static) `LogFormat` classes.
- All logging API is now thread-safe.
- Added support for custom log channels via generic class definition, i.e. `Logs.Get<YourLog>().Write(...)`, including editor integration.
- Moved log indentation state from the `Log` channel to the `ILogOutput` writer, so a writer subscribed to multiple different channels will still produce reasonable results when they differ in indentation.
- Reduced `LogEntry` struct to be plain-old-data (with the exception of the message string) and easily serializable when needed.
- Log timestamps are now in UTC.
- Introduced a specialized `EditorLogOutput` that supersedes the previous `InMemoryLogOutput`.
- Reworked visual logging API to mirror logging API, separating `VisualLog` and `VisualLogs`, as well as introducing custom log channel support via generic class definitions.

## Other

- Serializing a builtin .NET collection type will now omit the `_version` field, which is irrelevant for persistence, but can cause unnecessary merge conflicts.
- Plain old data structs such as `Vector2` now omit their contents during XML serialization when they equal their default value.
- Removed `CloneBehavior.WeakReference`, as it was superseded by explicit cloning and its related API additions. Internal cloning logic and public API related to weak references was simplified. This also fixes a bug where flagging an object reference as weak could affect the behavior of different references to the same objects.
- Renamed `CloneType` and related `ExtMethodsTypeInfo` API from `IsPlainOldData` to `IsCopyByAssignment` to reflect its usage and behavior more closely.
- Introduced `FileEvent` and `FileEventQueue` utility types to the core, making it easier to keep track of a sequence of file system changes.
- Updated input API for convenience, and to match other changes in the project.
- Reworked `Time` API to favor `DeltaTime` over `TimeMult`, increased convenience, and to match other changes in the project.
- Added list extension methods for zero-allocation stable sorting.
- Reworked `DualityUserData` and `DualityAppData` to match other changes in the project, and removed obsolete properties.
- The static `PhysicsUnit` and `AudioUnit` conversion classes now use static readonly fields instead of constants to allow future changes.
- Added `PinnedArrayHandle` helper struct to provide an easy way to communicate memory blocks to backend API.
- Optimized `RawList<T>` indexer to be inlineable more easily.
- Optimized `RawList<T>` clear and remove operations to not bother discarding internal data if it doesn't contain any object references.
- Added `RawListPool<T>` class to reduce per-frame allocations during heavy use of `RawList<T>` instances.
- Optimized reflection helper API to use static `<T>` lookups where possible.
- Optimized `MathF.NormalizeAngle` and `CircularDist` method implementations.
- Optimized `Rect.BoundingRadius` property implementation.
- Removed obsolete `Matrix4` and `Quaternion` API.