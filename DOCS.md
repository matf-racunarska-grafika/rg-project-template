# Project Documentation

This document explains how to set up, build and organize your project, and which coding conventions are expected. The conventions in sections 4 and 5 are part of the grade (**code quality**), so follow them from the first commit.

## Table of contents

0. [Installing dependencies](#0-installing-dependencies)
1. [Building from terminal](#1-building-from-terminal)
2. [Project structure](#2-project-structure)
3. [Resources](#3-resources)
4. [Programming guide](#4-programming-guide)
5. [Code style](#5-code-style)
6. [Troubleshooting](#6-troubleshooting)
7. [Working with Git](#7-working-with-git)

---

## 0. Installing dependencies

**Bundled in `libs/` (nothing to install):** GLFW, GLAD, GLM, spdlog, Dear ImGui, stb, nlohmann/json.

**Must be installed on the system:**

- A C++20 compiler and CMake >= 3.16
- OpenGL (headers/driver; ships with the OS or graphics driver, dev packages needed on Linux)
- Assimp

### Linux (Debian / Ubuntu)

```bash
sudo apt-get update
sudo apt install cmake git build-essential pkg-config \
    libgl1-mesa-dev libglvnd-dev mesa-common-dev mesa-utils \
    xorg-dev libx11-dev libxrandr-dev libxinerama-dev libxcursor-dev libxi-dev libxxf86vm-dev \
    libwayland-dev libxkbcommon-dev wayland-protocols \
    libassimp-dev
```

Optional: `doxygen graphviz` (docs), `clang-format` (formatting, see [section 5](#5-code-style)), `libfreetype-dev` (only if you use FreeType; picked up automatically when found).

> `libglfw3-dev`, `libglm-dev`, `libglew-dev` and `libsoil-dev` are **not** needed, because GLFW and GLM are built from `libs/`. If GLFW fails to configure on Wayland, add `-DGLFW_BUILD_WAYLAND=OFF` to the cmake configure command.

### Linux (Fedora)

```bash
sudo dnf install cmake git gcc-c++ pkgconf-pkg-config mesa-libGL-devel libglvnd-devel \
    libX11-devel libXrandr-devel libXinerama-devel libXcursor-devel libXi-devel libXext-devel \
    wayland-devel libxkbcommon-devel wayland-protocols-devel assimp-devel
```

### Linux (Arch)

```bash
sudo pacman -S cmake git base-devel pkgconf mesa libglvnd libx11 libxrandr libxinerama \
    libxcursor libxi wayland libxkbcommon wayland-protocols assimp
```

### macOS

```bash
xcode-select --install          # Clang, Apple SDK (OpenGL and Cocoa frameworks)
brew install cmake pkg-config assimp
```

- OpenGL ships with macOS (deprecated by Apple, maximum version is **4.1 core**).
- Your window creation code must request a core profile with forward compatibility, or context creation fails:

```cpp
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 1);
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
#ifdef __APPLE__
glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE);
#endif
```

- Shaders must use `#version 410 core` (or lower) on macOS. Compute shaders and 4.3+ features are not available.
- To keep one set of shaders for all platforms, use `#version 410 core` everywhere.

### Windows (Visual Studio + vcpkg, recommended)

1. Install [Visual Studio 2022](https://visualstudio.microsoft.com/) (or Build Tools) with the **Desktop development with C++** workload. This includes the Windows SDK (OpenGL headers and `opengl32.lib`) and a bundled CMake.
2. Install [Git](https://git-scm.com/download/win).
3. Install [vcpkg](https://github.com/microsoft/vcpkg) and Assimp:

```powershell
git clone https://github.com/microsoft/vcpkg C:\vcpkg
C:\vcpkg\bootstrap-vcpkg.bat
C:\vcpkg\vcpkg install assimp:x64-windows
```

4. Update your graphics drivers (OpenGL is provided by the GPU driver).

### Windows (MSYS2 / MinGW alternative)

Open the **MSYS2 UCRT64** shell:

```bash
pacman -S mingw-w64-ucrt-x86_64-toolchain mingw-w64-ucrt-x86_64-cmake \
    mingw-w64-ucrt-x86_64-ninja mingw-w64-ucrt-x86_64-assimp
```

### Verifying your setup

Check that the tools are available and recent enough:

```bash
cmake --version     # 3.16 or newer
g++ --version       # or: clang++ --version / cl (MSVC); must support C++20
```

---

## 1. Building from terminal

The executable is named `project` (`project.exe` on Windows) and is placed in the project root, so relative resource paths (`resources/...`) work when you run it from there.

### Linux / macOS (and MSYS2)

For a standard build:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
./project
```

For separate release and debug builds:

```bash
cmake -S . -B cmake-build-release -DCMAKE_BUILD_TYPE=Release
cmake --build cmake-build-release --parallel
./project
```

```bash
cmake -S . -B cmake-build-debug -DCMAKE_BUILD_TYPE=Debug
cmake --build cmake-build-debug --parallel
./project
```

- **Release** build: optimizations on, best performance. Use it for demos and the final presentation.
- **Debug** build: optimizations off, slower, but much better debugging support (breakpoints, variable inspection). Use it while developing.

On MSYS2, pass `-G Ninja` to the first `cmake` command to use the installed Ninja generator.

If CMake cannot find Assimp on macOS with Homebrew:

```bash
cmake -S . -B build -DCMAKE_PREFIX_PATH="$(brew --prefix)"
```

### Windows (Visual Studio generator + vcpkg)

```powershell
cmake -S . -B build -DCMAKE_TOOLCHAIN_FILE=C:/vcpkg/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release --parallel
.\project.exe
```

vcpkg copies the Assimp DLLs next to the executable automatically. You can also open the `build` folder in Visual Studio (`build\project.sln`); `project` is set as the startup project.

Visual Studio and Xcode are multi-config generators: choose the configuration with `--config Debug|Release` at build time. For single-config generators (Makefiles, Ninja) use `-DCMAKE_BUILD_TYPE`. If none is given, `Release` is used.

### Running from an IDE

If you use CLion, VS Code, Visual Studio or another IDE, set the **working directory to the project root**. Otherwise the program will not find `resources/shaders`, `resources/objects` and `resources/textures`.

### Starting over

If the configuration gets into a strange state (moved folder, changed compiler, new dependency), delete the build folder and configure again:

```bash
rm -rf build
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
```

---

## 2. Project structure

```
.
├── app/                 # Your application source code
├── cmake/modules/       # CMake find modules (e.g. for Assimp)
├── libs/                # Bundled third-party libraries (do not edit)
├── resources/           # Runtime assets, see section 3
│   ├── shaders/
│   ├── objects/
│   └── textures/
├── .clang-format        # Formatting rules
├── .gitattributes
├── .gitignore
├── CMakeLists.txt       # Build script, produces the `project` executable
├── DOCS.md              # This document
├── LICENSE
├── README.md
└── vcpkg.json           # Dependency manifest used with vcpkg on Windows
```

Rules of thumb:

- Write your code in `app/`. Do not modify `libs/`.
- Do not commit build output (`build/`, `cmake-build-*/`, the `project` executable). These should already be covered by `.gitignore`.
- Keep assets only in `resources/` so the repository stays organized and the program can always find them.

### Suggested layout inside `app/`

This is a recommendation, not a requirement. Adapt it to your project, but keep logical units in separate files (see the [programming guide](#4-programming-guide)).

```
app/
├── main.cpp             # Entry point: create context, run main loop
├── app_context.hpp      # AppContext: application state (time, input, camera, scene objects)
├── gfx_context.hpp      # GFXContext: OpenGL state (shaders, framebuffers, textures)
├── platform/            # Window creation, input callbacks
├── renderer/            # Draw passes: geometry, lighting, post-processing, skybox
├── scene/               # Objects, lights, camera, event sequence
├── gui/                 # ImGui panels (lighting configuration, debug info)
└── utils/               # Shader/texture/model loading, math helpers, logging
```

---

## 3. Resources

All assets that the program loads at runtime live under `resources/`, in exactly three subfolders:

| Folder | Contents | Examples |
|---|---|---|
| `resources/shaders/` | GLSL shader source files | `blinn_phong.vs`, `blinn_phong.fs`, `skybox.vs`, `skybox.fs`, `bloom_blur.fs` |
| `resources/objects/` | 3D models and the files they depend on (material files, and textures that belong to a single model) | `backpack/backpack.obj`, `lamp/lamp.gltf` |
| `resources/textures/` | Standalone images used directly by the code | `wood_diffuse.png`, `wood_specular.png`, `skybox/right.jpg` |

### Conventions

- Paths in code are **relative to the project root**, e.g. `resources/shaders/blinn_phong.fs`. This is why the program must be started from the project root.
- Use `snake_case` lowercase file names without spaces, and forward slashes in paths (they work on every platform).
- Give each model its own subfolder under `resources/objects/`, for example `resources/objects/backpack/`, and keep its material files and model-specific textures next to it. Assimp resolves texture paths relative to the model file.
- Group multi-file textures in a subfolder, e.g. the six cubemap faces in `resources/textures/skybox/` (`right`, `left`, `top`, `bottom`, `front`, `back`).
- Name texture maps with a suffix that says what they are: `_diffuse`, `_specular`, `_normal`, `_height`.
- Name shaders after what they do and use one pair per effect. Keep the extension consistent across the project (for example `.vs` / `.fs` / `.gs`, or `.vert` / `.frag` / `.geom`).
- Check the license of every downloaded asset and keep attribution (author, source link) in a text file next to it, or in a `resources/CREDITS.md`.
- Keep the repository light: compress large textures, and avoid committing multi-hundred-megabyte models.

---

## 4. Programming guide

- Manage application state via one global `AppContext` object
- Manage graphics/OpenGL state via one global `GFXContext` object
- Variable names should be clear and descriptive
- Keep the code modular: structs, functions and classes that belong to logical units should be in their own file (math, renderer, platform, utils...)
- Functions should have clear verb names that describe what the function does
- Put project code inside a namespace
- Use new/delete over malloc/free
- Use smart pointers over new/delete
- Use standard containers (vector, unordered_set, map, set...) when appropriate
- Use standard algorithm header when appropriate

### Additional recommendations

- **One responsibility per function.** If a function needs a comment block to explain its sections, split it. The main loop should read like a table of contents: `process_input()`, `update()`, `render()`.
- **Avoid magic numbers.** Name constants (`constexpr float CAMERA_SPEED = 2.5f;`) instead of repeating literals.
- **Prefer references and `const`.** Pass big objects by `const&`, mark functions and variables `const` when they do not change.
- **Check for errors.** Log failed shader compilation, missing files and failed model loading with `spdlog` instead of failing silently.
- **Use `spdlog` for logging**, not `std::cout` / `printf`.
- **Keep headers light.** Include only what you need, and prefer forward declarations in headers when possible.
- **Delete dead code.** Do not leave large blocks of commented-out code in commits; Git keeps the history.
- **Comment the why, not the what.** Explain non-obvious decisions, formulas and OpenGL state requirements, not what each line obviously does.

---

## 5. Code style

### Naming conventions

| Element | Style | Example |
|---|---|---|
| Global variables | `g_` prefix | `g_counter` |
| Namespaces | snake_case | `math_utils` |
| Structs | PascalCase | `Point3D` |
| Classes | PascalCase | `Calculator` |
| Static / constant variables | UPPER_CASE | `MAX_OPERATIONS` |
| Public member variables | snake_case | `is_active` |
| Protected member variables | `m_` prefix + snake_case | `m_name` |
| Private member variables | `m_` prefix + snake_case | `m_max_size` |
| Member functions | snake_case | `calculate_average` |
| Free functions | snake_case | `process_data` |
| Parameters | snake_case | `max_size` |
| Local variables | snake_case | `initial_capacity` |
| Template parameters | PascalCase | `Container`, `Result`, `MaxSize` |

### Formatting rules

- Control flow statements (`if` / `for` / `while` / `switch` / `try`) always with braces, even for a single statement
- The opening brace stays on the same line as the statement; statements within braces always start on a new line
- One line per variable declaration, no multiple variable declarations on a single line
- Use the `.clang-format` file from the repository root. Most editors can apply it automatically on save.

### Formatting automatically

If `clang-format` is installed, format a file in place with:

```bash
clang-format -i app/path/to/file.cpp
```

or all project sources at once (Linux / macOS):

```bash
find app -name '*.cpp' -o -name '*.hpp' -o -name '*.h' | xargs clang-format -i
```

Format before every commit, so that style changes never mix with logic changes in the history.

### Example

```cpp
#include <string>
#include <vector>

// Global variable with g_ prefix
int g_counter = 0;

// Namespace in snake_case
namespace math_utils {

// Struct in PascalCase
struct Point3D {
    float x;
    float y;
    float z;
};

// Class in PascalCase
class Calculator {
public:
    // Static and/or constant in UPPER_CASE
    static const int MAX_OPERATIONS = 100;

    // Public member variable in snake_case
    bool is_active;

    // Constructor (parameters in snake_case).
    // Initializer list follows the declaration order of the members.
    Calculator(int max_size)
        : is_active(true)
        , m_max_size(max_size) {
        // Local variable in snake_case
        int initial_capacity = max_size * 2;
        m_results.reserve(initial_capacity);
    }

    // Member function in snake_case
    float calculate_average(const std::vector<float>& values) {
        // Parameter and local variables in snake_case,
        // one declaration per line
        float sum = 0.0f;
        int count = 0;

        // Braces always, opening brace on the same line
        for (const auto& value : values) {
            sum += value;
            ++count;
        }

        if (count > 0) {
            return sum / count;
        }

        return 0.0f;
    }

protected:
    // Protected member with m_ prefix
    std::string m_name;

private:
    // Private member variables with m_ prefix
    int m_max_size;
    std::vector<float> m_results;
};

} // namespace math_utils

// Free function in snake_case
void process_data() {
    // Local variable in snake_case
    math_utils::Calculator calc(50);
    ++g_counter;
}

// Template parameters always in PascalCase
template<typename Container, typename Result, int MaxSize>
Result operation(const Container& c) {
    // ...
}
```

### Code review checklist

Before submitting, go through this list:

- [ ] Code is formatted with `clang-format`
- [ ] Names follow the conventions above and describe what the thing is or does
- [ ] Project code is inside a namespace, and logical units are in their own files
- [ ] No leftover debug prints, commented-out code or unused files
- [ ] No hard-coded absolute paths; all assets are loaded from `resources/`
- [ ] The project builds from a clean checkout in both Debug and Release
- [ ] The program starts from the project root and shows the scene without errors in the log

---

## 6. Troubleshooting

| Problem | Fix |
|---|---|
| `Could NOT find OpenGL` (Linux) | Install `libgl1-mesa-dev` and `libglvnd-dev` |
| GLFW: missing X11/Wayland headers | Install the `xorg-dev` / `libwayland-dev` / `libxkbcommon-dev` / `wayland-protocols` packages from section 0 |
| GLFW configure fails on Wayland | Add `-DGLFW_BUILD_WAYLAND=OFF` to the cmake configure command |
| `Could NOT find ASSIMP` | Install Assimp (section 0). On macOS/Windows pass `CMAKE_PREFIX_PATH` or the vcpkg toolchain file |
| Black window / context failure on macOS | Use the core profile + forward-compat hint and GLSL `#version 410 core` |
| `assimp-vc143-mt.dll` not found (Windows) | Build with the vcpkg toolchain file, or copy the DLL next to `project.exe` |
| Shader, texture or model fails to load | Run the program from the project root (or set the IDE working directory to it) and check the path against `resources/shaders`, `resources/objects` and `resources/textures` |
| Model loads but is untextured | Check that the textures referenced by the model file are present next to it in `resources/objects/<model>/` |
| Shader compiles on one machine but not another | Use `#version 410 core` and avoid features above OpenGL 4.1; some drivers are more lenient than others |
| Stale build after pulling or moving the project | Delete the build folder and run CMake again |

---

## 7. Working with Git

- Commit small, logical changes with clear messages, for example `Add point light to lighting system` rather than `update`.
- Push regularly, so your work is backed up and visible.
- Do not commit build folders, executables or IDE settings. If something unwanted shows up in `git status`, add it to `.gitignore`.
- Large binary assets (textures, models) are tracked in the repository too. Check `.gitattributes`, and avoid replacing the same large file many times, since every version stays in the history.
- Keep formatting-only changes in separate commits from functional changes.
