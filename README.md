# CRYENGINE 5.7.1 — Asset Browser Thumbnail Improvements

Improves thumbnail generation for Mesh, Skinned Mesh, Skeleton, and Material assets in CRYENGINE 5.7.1 Sandbox.

## Changes

* Fixed thumbnails capturing the previous asset due to the multithreaded renderer's one-frame delay.
* Added environment lighting and reflections using the original EngineAssets.
* Waits for full-resolution textures before capture, with timeout and stall fallbacks.
* Improved camera framing using bounding-box fitting.
* Added a scalable grey studio background.

These changes only affect thumbnail generation. Interactive preview windows remain unchanged.

## Modified Files

```text
Code/Sandbox/EditorQt/QT/Widgets/QPreviewWidget.cpp
Code/Sandbox/EditorQt/QT/Widgets/QPreviewWidget.h
Code/Sandbox/EditorQt/AssetSystem/MeshType.cpp
Code/Sandbox/EditorQt/AssetSystem/SkinnedMeshType.cpp
Code/Sandbox/EditorQt/AssetSystem/SkeletonType.cpp
Code/Sandbox/EditorQt/AssetSystem/MaterialType.cpp
```

## Installation

From the CRYENGINE source root:

```bash
git apply --check patches/cryengine-5.7.1-thumbnail-fix.patch
git apply patches/cryengine-5.7.1-thumbnail-fix.patch
```

For files with existing modifications:

```bash
git apply --3way patches/cryengine-5.7.1-thumbnail-fix.patch
```

Alternatively, copy the repository's `Code/` directory into the CRYENGINE source tree. Only do this if the affected files have not been modified.

Visual settings can be adjusted in the `Private_PreviewWidget` namespace near the top of `QPreviewWidget.cpp`.
