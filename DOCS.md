# Project Documentation

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

Optional: `doxygen graphviz` (docs), `libfreetype-dev` (only if you use FreeType; picked up automatically when found).

_Note: `libglfw3-dev`, `libglm-dev`, `libglew-dev` and `libsoil-dev` are NOT needed, because GLFW and GLM are built from `libs/`. If GLFW fails to configure on Wayland, add `-DGLFW_BUILD_WAYLAND=OFF` to the cmake configure command._

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

## 1. Building from Terminal

The executable is named `project` (`project.exe` on Windows) and is placed in the project root, so relative resource paths (`resources/...`) work when you run it from there. Shader files in `resources/shaders`.

### Linux / macOS (and MSYS2)

For a standard build:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
./project
```

For having separate release and debug builds:

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

Release build - optimizations turned on => optimal performance
Debug build - optimizations turned off => worse performance, better debugging support

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

### Troubleshooting

| Problem                                   | Fix                                                                                                          |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `Could NOT find OpenGL` (Linux)           | Install `libgl1-mesa-dev` and `libglvnd-dev`                                                                 |
| GLFW: missing X11/Wayland headers         | Install the `xorg-dev` / `libwayland-dev` / `libxkbcommon-dev` / `wayland-protocols` packages from section 0 |
| `Could NOT find ASSIMP`                   | Install Assimp (section 0). On macOS/Windows pass `CMAKE_PREFIX_PATH` or the vcpkg toolchain file            |
| Black window / context failure on macOS   | Use the core profile + forward-compat hint and GLSL `#version 410 core`                                      |
| `assimp-vc143-mt.dll` not found (Windows) | Build with the vcpkg toolchain file, or copy the DLL next to `project.exe`                                   |

## 2. Programming guide

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

## 3. Code style

Naming Conventions:

- Global variables: g\_ prefix (e.g., g_counter)
- Namespaces: snake_case (e.g., math_utils)
- Structs: PascalCase (e.g., Point3D)
- Classes: PascalCase (e.g., Calculator)
- Static/constant variables: UPPER_CASE (e.g., MAX_OPERATIONS)
- Public member variables: snake_case (e.g., is_active)
- Protected member variables: m\_ prefix + snake_case (e.g., m_name)
- Private member variables: m\_ prefix + snake_case (e.g., m_max_size)
- Member functions: snake_case (e.g., calculate_average)
- Free functions: snake_case (e.g., process_data)
- Parameters: snake_case (e.g., max_size)
- Local variables: snake_case (e.g., initial_capacity)
- Template parameters: PascalCase (e.g., Container, Result, MaxSize)
- Control flow statements (if/for/while/switch/try) always with braces
- Statement within braces always on a new line from the brace
- One line per variable declaration, no multiple variable declarations on a single line

```C++
#include <vector>
#include <string>

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

    // Constructor (parameters in snake_case)
    Calculator(int max_size) : m_max_size(max_size), is_active(true) {
        // Local variable in snake_case
        int initial_capacity = max_size * 2;
        m_results.reserve(initial_capacity);
    }

    // Member function in snake_case
    float calculate_average(const std::vector<float>& values) {
        // Parameter and local variables in snake_case
        float sum = 0.0f;
        int count = 0;

        // Loop with braces on same line
        for (const auto& value : values) {
            sum += value;
            ++count;
        }

        // If statement with braces on same line
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
   // .
}
```
