# RG Project Template

Starter template for the final project in the *Computer Graphics* (Računarska grafika) course at MATF, University of Belgrade. It gives you a ready-to-build C++20 / OpenGL project, so you can spend your time on graphics instead of build setup.

> **Everything about installing, building, coding conventions and code style is in [DOCS.md](DOCS.md). Read it before you write any code.**

> **Before submitting the final project replace the contents of this file with the description of your project**  

## Table of contents

1. [Getting started](#getting-started)
2. [Repository structure](#repository-structure)
3. [What should the project contain?](#what-should-the-project-contain)
4. [Tips for the requirements](#tips-for-the-requirements)
5. [Documentation](#documentation)
6. [License](#license)

---

## Getting started

1. **Create your own repository from this template.** Click **Use this template** on the GitHub page of this repository (or fork it), then clone *your* copy:

   ```bash
   git clone https://github.com/<your-username>/<your-project>.git
   cd <your-project>
   ```

2. **Install dependencies** for your platform, see [DOCS.md → Installing dependencies](DOCS.md#0-installing-dependencies). In short, you need a C++20 compiler, CMake 3.16+, OpenGL development files and Assimp. Everything else is bundled.

3. **Build and run** (Linux / macOS / MSYS2):

   ```bash
   cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
   cmake --build build --parallel
   ./project
   ```

   On Windows with Visual Studio and vcpkg:

   ```powershell
   cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=C:/vcpkg/scripts/buildsystems/vcpkg.cmake
   cmake --build build --config Release --parallel
   .\project.exe
   ```

   The executable is placed in the project root and must be started from there, so that relative paths such as `resources/shaders` work. See [DOCS.md → Building from Terminal](DOCS.md#1-building-from-terminal) for Debug builds and troubleshooting.

---

## Repository structure

```
.
├── app/                # Your application source code
├── cmake/modules/      # CMake find modules
├── libs/               # Bundled libraries (nothing to install):
│                       #   GLFW, GLAD, GLM, spdlog, Dear ImGui, stb, nlohmann/json
├── resources/          # All runtime assets
│   ├── shaders/        #   GLSL shader files (.vs, .fs, .gs, ...)
│   ├── objects/        #   3D models (.obj, .fbx, .gltf, ...) and their material files
│   └── textures/       #   Images: diffuse/specular/normal maps, skybox faces, ...
├── .clang-format       # Formatting rules
├── CMakeLists.txt      # Build script, produces the `project` executable
├── vcpkg.json          # Dependency manifest used on Windows
├── DOCS.md             # Setup, build, programming guide and code style
└── LICENSE             # CC0 1.0
```

Only **Assimp** (model loading) and OpenGL development files have to be installed on your system.

---

## What should the project contain?

Use the checklists below to track your progress. Copy them into your own README and tick items as you complete them.

### Core requirements (mandatory)

- [ ] A 3D world
- [ ] A movable camera
- [ ] Blending
- [ ] Face culling
- [ ] Cubemaps
- [ ] Instancing *(to be confirmed by the instructors)*
- [ ] Blinn-Phong lighting model applied to all objects
- [ ] A lighting system supporting all three types of lights (`Directional`, `Point`, `Spot`)
- [ ] Lighting that can be configured through a graphical user interface (ImGui)
- [ ] Offscreen image post-processing
- [ ] An implemented sequence of events:
  - `{ACTION_A1}` → **AFTER_M_SECONDS** → `{EVENT_X}` → **AFTER_N_SECONDS** → `{EVENT_Y}`
  - **ACTION**: moving the camera to a location in the scene, reaching a specific point in time, etc.
  - **AFTER_X_SECONDS**: after X seconds have elapsed since the registered action
  - **EVENT**: something moves in the scene, a light changes color, an object disappears, an object appears, etc.

### Additional features (bonus points)

- [ ] **(10 points)** [Bloom](https://learnopengl.com/Advanced-Lighting/Bloom) or [Point Shadows](https://learnopengl.com/Advanced-Lighting/Shadows/Point-Shadows)
- [ ] **(15 points)** [Deferred Shading](https://learnopengl.com/Advanced-Lighting/Deferred-Shading)

### Also graded

- [ ] **Visibility, prominence, and contribution** of the implemented features to the atmosphere of the scene
- [ ] **Code quality**
  - [ ] Consistent formatting according to the instructions in [DOCS.md](DOCS.md#5-code-style)
  - [ ] Modularity and logical organization of the code
  - [ ] Clear and understandable names for classes, functions, and variables

---

## Tips for the requirements

How the template and its documentation relate to the points above:

| Requirement | Relevant notes |
|---|---|
| **Code quality, formatting** | Naming conventions and brace style are in [DOCS.md → Code style](DOCS.md#5-code-style) (`g_` prefix for globals, `snake_case` functions and variables, `PascalCase` types, `m_` prefix for member variables, braces on all control flow). The repository includes a `.clang-format` file, so configure your editor to use it. |
| **Code quality, modularity** | Follow [DOCS.md → Programming guide](DOCS.md#4-programming-guide): one `AppContext` for application state, one `GFXContext` for OpenGL state, separate files for logical units (math, renderer, platform, utils), project code in a namespace, smart pointers and standard containers over raw memory management. |
| **ImGui lighting controls** | Dear ImGui is already bundled in `libs/`, nothing to install. |
| **Models in the 3D world** | Loading goes through Assimp, which is the one library you install yourself. Put models in `resources/objects`. |
| **Shaders** | Keep all shader files in `resources/shaders`. |
| **Textures and cubemaps** | Put images, including the six skybox faces, in `resources/textures`. See [DOCS.md → Resources](DOCS.md#3-resources) for naming suggestions. |
| **macOS** | OpenGL is limited to **4.1 core** and shaders must start with `#version 410 core`. Create the window with the core profile and forward-compat hints shown in [DOCS.md](DOCS.md#macos). Compute shaders and 4.3+ features are unavailable, which is fine for every feature listed above, including bloom, point shadows and deferred shading. |
| **Offscreen post-processing, bloom, deferred shading** | These all build on framebuffers (including multiple render targets for deferred shading and HDR for bloom), covered in the *Framebuffers*, *HDR*, *Bloom* and *Deferred Shading* chapters of [learnopengl.com](https://learnopengl.com). The [course LearnOpenGL fork](https://github.com/matf-racunarska-grafika/LearnOpenGL) has working samples for each. |
| **Blending, face culling, cubemaps, instancing, Blinn-Phong, light types** | Reference samples exist in the same fork under `4.advanced_opengl`, `2.lighting` and `5.advanced_lighting`. |
| **Sequence of events** | Keep the event system independent from rendering: a small timeline in `AppContext` that stores registered actions, timestamps and callbacks makes events easy to test and extend. |
| **Visibility and atmosphere** | Make each feature clearly visible in the final scene: place lights where their effect can be seen, show blending with an actual transparent object, use the skybox to match your mood. Features that cannot be noticed contribute little to the grade. |

---

## Documentation

| Topic | Where |
|---|---|
| Installing dependencies (Linux, macOS, Windows) | [DOCS.md §0](DOCS.md#0-installing-dependencies) |
| Building from the terminal, Debug vs Release | [DOCS.md §1](DOCS.md#1-building-from-terminal) |
| Project structure | [DOCS.md §2](DOCS.md#2-project-structure) |
| Resources: shaders, objects, textures | [DOCS.md §3](DOCS.md#3-resources) |
| Programming guide | [DOCS.md §4](DOCS.md#4-programming-guide) |
| Code style and naming conventions | [DOCS.md §5](DOCS.md#5-code-style) |
| Troubleshooting common build errors | [DOCS.md §6](DOCS.md#6-troubleshooting) |
| Git workflow | [DOCS.md §7](DOCS.md#7-working-with-git) |

Learning material: [learnopengl.com](https://learnopengl.com) and the [course LearnOpenGL fork](https://github.com/matf-racunarska-grafika/LearnOpenGL).

---

## License

This template is released under [CC0 1.0 Universal](LICENSE). Third-party libraries in `libs/` keep their own licenses.
