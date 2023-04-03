# GSoC 2023 OpenSCAD Proposal: OpenGL Modernization

## Project Information

OpenSCAD mainly still relies on legacy "fixed-function" aka "immediate mode" OpenGL to render the user's 3D model. With this older method, the vertices and shading color of surfaces are specified on an individual triangle basis.

Other parts of the OpenSCAD codebase provide the groundwork to instead use the newer approach of utilizing Vertex Buffer Objects (VBOs) to hold data that the GPU will use to render. This modern method enables the use of programmable GLSL shaders that can dynamically alter the appearance of objects. My 2022 GSoC project added support for these shaders. Still, there is a large set of additional work required in order to enable a robust rollout of these new capabilities.

| Project Title |
| --- |
| OpenGL Modernization |

## Project Summary

The project would undertake the necessary code restructuring to facilitate OpenSCAD's transition to rendering with modern OpenGL. An important early step will be adding clear delineation between code paths that result in underlying calls to the two different rendering methods. This groundwork would then grow into developer-controlled flags that would enable the eventual deprecation of the legacy fixed-function calls. Part of the process will involve assessing the readability and testability of higher-level rendering classes and performing refactoring in line with guidance from other developers. Another key area of focus includes assessing and implementing any remaining workarounds needed to replicate the information exchanged between OpenCSG and the legacy rendering classes.

## Detailed Description

The process for rendering in OpenSCAD consists of a series of stages, some handled by external libraries. Abstractly, the process of transforming the textual SCAD code into geometry can be treated as the first stage, and the process of rendering the geometry is the second. Depending on the configuration, different libraries and APIs might be used for these stages.

With a typical configuration, generally speaking the F5 preview mode will use `OpenCSGRenderer` whereas the F6 render will use `CGALRenderer`.

Currently, the code in `master` will use legacy OpenGL except when all of the following are true:

* A development build is running
* The Experimental VxO features are enabled (these checkboxes are in the settings menu)
* Show Edges is enabled
* An F5 preview is being rendered

When those conditions are met, the following things happen in the underlying code:

1. `OpenCSGRenderer` is being used instead of `CGALRenderer`
2. `shaderinfo` is non-null
3. `PolySet`s are being used instead of other geometry structures

The changes introduced by [#4330](https://github.com/openscad/openscad/pull/4330/) allow modern OpenGL shaders to be used in the F6 render. Key pieces of how this was accomplished can be broadly summarized as:

1. Copy the modern-OpenGL-invoking code from `OpenCSGRenderer` to `CGALRenderer`
2. Set `shaderinfo` to the user's selected shader
3. Convert all the generated geometry to a `PolySet`, if a different type of object was generated instead

Note how these correspond to the other numbered list of conditions above them. These three areas will form the basis for refactoring the rendering classes. Currently, it isn't possible to identify individual sections of code that always accomplish the same step in the rendering process. Variables such as `shaderinfo` are passed through a chain of methods, presenting a barrier to readability and testing. Moreover, the process of switching to modern OpenGL will require swapping classes for new implementations--a task that is much harder when their references are scattered.

It's also worth noting that even when VBOs are used, the existing codebase might still perform other legacy API calls. I plan to evaluate the success of OpenGL modernization progress by attempting to run a build made with the OpenGL 3+ Core Profile.

After refactoring to consolidate rendering operations, controls would be added to select which set of implementations to use. This phase will particularly benefit from consultation with other developers about the future outlook for each of the current libraries in use. For example, parts of both the legacy and modern OpenGL rendering methods hinge on the availability of OpenCSG and/or CGAL. Certain combinations of (de)activated features may require additional conversions or workarounds to function.

One important thing to track during this process is the initial source of prerequisite information for rendering. Changes to the set of libraries in use could prevent certain visual features from working unless extra effort is made to preserve them. A motivating example is that the changes introduced by [#4330](https://github.com/openscad/openscad/pull/4330/) required a workaround to keep track of marked faces. That information was previously given by CGAL geometry structures, but got lost when converting to a `PolySet`. As of right now, the 4330 pull request adds a field to the PolySet implementation that stores a `marked` attribute that is passed to the shader as a vertex attribute. A similar approach may be necessary again.

An additional component of modernization plans is preparing to target other platforms, by methods such as providing GLES support. Both the included and user-provided shaders would be validated for compatibility with the feature sets of relevant OpenGL versions.

These changes should all be planned in the context of other ongoing OpenGL improvements. Many of them, such as [new error macros](https://github.com/openscad/openscad/pull/4570) and [other OpenGL refactoring](https://github.com/openscad/openscad/pull/4576) occur elsewhere in the codebase. But coordination may still be required in the event that dependent code is changed during development.

## Planned Resources

The relevant libraries and frameworks for this project are already dependencies for other parts of the OpenSCAD codebase. Here are the main tools:

- [OpenGL](https://www.khronos.org/opengl/)
- [Qt](https://code.qt.io/cgit/)
- [CGAL](https://www.cgal.org/)
- [OpenCSG](https://opencsg.org/)

## Deliverables

The final result will include a set of changes to modernize OpenSCAD's OpenGL rendering process. This will take the form of GitHub pull request(s) which contain all the additional classes, resources, tests, and documentation required to integrate the changes into the codebase.

Specifically, as outlined in the project description, my contributions would include refactoring to support transitioning to modern OpenGL, and a set of controls to define which rendering methods and libraries should be invoked.

## Development Schedule

| Date | Event |
| --- | --- |
| May 29 | Coding Begins |
| June 23 | Milestone 1: Refactoring for stage separation in rendering |
| July 14 | Milestone 2: Official Midterm Evaluation deadline; "Is it on track towards modern OpenGL support across viewport modes?" |
| August 8 | Milestone 3: Developer controls for library and API use |
| September 5 | Final Week: Code, documentation, and tests completed |
