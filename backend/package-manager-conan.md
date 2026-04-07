# C/C++ Package Manager: Conan

## Introduction

A package manager is a tool that automates the management of project dependencies, their installation, updating, configuration, and integration into the build process.

Historically, C and C++ never developed a single standard package manager the way more modern languages did, such as JavaScript (npm) or Python (pip). This is because C/C++ lacks a standard build system instead of a unified tool, various solutions are used, such as CMake, Make, MSBuild, and Ninja. Historical legacy also plays a role, C and C++ appeared long before modern approaches to dependency management, so libraries were traditionally distributed through system managers or added manually.

### The Conan Package Manager

Conan is a dependency and package manager for C and C++. It is free, [open source](https://github.com/conan-io/conan), distributed under the MIT license, and runs on most platforms (Windows, Linux, macOS, and others). It can be used to develop for a wide range of target platforms and integrates with popular build systems such as CMake, Visual Studio (MSBuild), Makefiles, SCons, and others.

Conan lets developers describe the required libraries and their versions in a dedicated configuration file called `conanfile.py`, after which it independently handles downloading packages, resolves their dependencies, and when necessary builds them from source code, taking into account the specific environment including compiler, architecture, and build parameters.

## How Conan Works with Packages

Conan uses a decentralized package management model. This means that dependencies can be downloaded from various remote repositories (remotes) and can also be published independently, similar to how Git works.

Each package in Conan consists of two key parts:
- **Recipe** — a description of how to obtain and build the package (`conanfile.py`)
- **Binary package** — a built artifact that depends on a specific configuration (compiler, architecture, build type, etc.)

When installing dependencies, Conan:
1. Builds the project's dependency graph.
2. Checks for suitable binary packages in the local cache.
3. Downloads them from remote servers if needed.
4. If no binary is found, builds the package from source.

All recipes and binary packages are stored in the local Conan cache. You can find its path with:
```bash
conan config home
```

The cache location can be overridden using an environment variable:
```bash
export CONAN_HOME=./.conan2
```

## Installing Conan

Conan is an external dependency management tool. It is installed into the system or a virtual environment and can then be used across any projects.

Unlike libraries, Conan is not added to a project as a dependency it is used as a CLI tool for managing builds and dependencies.

There are several ways to install Conan:
1. Using the pip package manager (the most preferred and strongly recommended method).
2. Using a system installer to avoid requiring Python.

### Using the pip Package Manager

pip is the standard package manager for Python libraries and modules. It allows you to easily install, update, and remove third-party libraries from the [PyPI (Python Package Index)](https://pypi.org/) repository. If needed, pip can be installed by following the instructions in the [documentation](https://pip.pypa.io/en/stable/installation/).

Conan is available as a [package](https://pypi.org/project/conan/) on PyPI, because Conan itself is implemented in Python. Installing the latest version of Conan via pip requires Python version >= 3.8.

To install Conan, run the following command:
```bash
pip install conan
```

Once the command completes, Conan will be installed on your system.

### Installing via a System Installer

In addition to pip, Conan is also distributed as ready-made binary builds intended for system-level installation without requiring Python. The corresponding installers are available for download from the [official GitHub repository](https://github.com/conan-io/conan/releases/).

After downloading the installer, run it. Conan will be installed on your system.

### Verifying the Conan Installation

To verify that the installation was successful, run:
```bash
conan version
```

If the installation was successful, this command will display the installed version of Conan.

## Configuring Conan

### Conanfile

To use Conan in a project, you need to create a `conanfile.py` the main configuration file that describes the project's dependencies and the rules for working with them. It specifies the required libraries and their versions, and can also define build parameters, environment settings, and instructions for integrating with the build system (e.g., CMake).

A minimal `conanfile.py` might look like this:
```python
from conan import ConanFile
from conan.tools.cmake import CMakeDeps, CMakeToolchain

class ExampleProject(ConanFile):
    name = "example-project"  # Project/recipe name
    version = "0.0.1"         # Project/recipe version

    # Dependency declaration block
    def requirements(self):
        self.requires("drogon/1.9.12")  # Web framework (example)
        self.requires("gtest/1.14.0")   # Testing library (example)

    # Build system configuration block (CMake used as example)
    def generate(self):
        # CMakeDeps is a generator that creates Find<Package>.cmake files for each
        # dependency so that CMake can locate them via find_package()
        deps = CMakeDeps(self)
        deps.generate()

        # CMakeToolchain is a generator that creates a conan_toolchain.cmake file
        # containing compiler settings, flags, and variables from the Conan profile
        tc = CMakeToolchain(self)
        tc.generate()
```

> **NOTE:** The `conanfile.py` is also referred to as a recipe, because it describes how to obtain and build a package.

> **NOTE:** The `conanfile.py` is typically placed in the root of the project.

> **NOTE:** A simpler alternative, `conanfile.txt`, can also be used a format without any logic, intended for basic scenarios.

More information about `conanfile.py` can be found in the [official Conan documentation](https://docs.conan.io/2/reference/conanfile.html).

### Profiles

In addition to `conanfile.py`, Conan uses profiles special configuration files that let you define a complete set of environment parameters, including compiler and build settings, custom options, environment variables (for both the build and run stages), tool requirements, and additional configuration variables. This brings everything together in one place for convenient and reproducible project configuration management.

There are two types of profiles:
- **Build profile** — describes the characteristics of the machine on which the application will be built.
- **Host profile** — describes the characteristics of the machine on which the application will run.

The distinction between Build and Host profiles is especially important during cross-compilation, for example, when building on `x86_64` but targeting `arm64`.

A minimal Conan profile might look like this:
```ini
[settings]
os=Linux                      # Operating system
arch=x86_64                   # System architecture
build_type=Debug              # Build type
compiler=clang                # Compiler being used
compiler.version=14           # Compiler version
compiler.libcxx=libstdc++11   # Standard library implementation (GCC, C++11 ABI)
compiler.cppstd=gnu20         # C++ standard version with GNU extensions

[conf]
# Paths to compiler executables
tools.build:compiler_executables={"c":"clang","cpp":"clang++"}
# Number of parallel threads during build
tools.build:jobs=6
# System package manager mode (install — install packages automatically)
tools.system.package_manager:mode=install
```

To apply a profile, specify it explicitly when running a command using the `--profile:build` (`-pr:b`) or `--profile:host` (`-pr:h`) flags for the Build and Host profiles respectively, or `--profile:all` (`-pr:a`) to apply the same profile to both the Build and Host contexts simultaneously.

For example, when running the command that installs project dependencies:
```bash
conan install . --build=missing --profile:all=./profiles/conan-example-profile.profile
```
where `--build=missing` means that if Conan cannot find a pre-built binary for the current project configuration, it should build it itself.

The argument accepts both relative and absolute paths.

More information about profiles can be found in the [official Conan documentation](https://docs.conan.io/2/reference/config_files/profiles.html).

#### Default Profile

If no profile is specified at the time of invocation, the default profile defined in the Conan home directory will be used.

The default profile is located at `.conan2/profiles/default`, for example, `~/.conan2/profiles/default`. To view its contents, run `conan profile show default`.

The `default` profile may not exist initially. In that case, it must be generated with:
```bash
conan profile detect
```

This command automatically detects system parameters such as the operating system, compiler and its version, standard library, and architecture.

#### The Jinja2 Templating Engine

A templating engine is typically a library that automatically generates text documents by combining static templates with dynamic data.

Jinja2 is a templating engine written in Python. It allows you to create templates containing special placeholders in which you can write code using Python-like syntax. The document is then rendered, with placeholders replaced by actual data.

Conan supports the use of the Jinja2 templating engine in configuration files, including profiles. This allows parameter values to be dynamically determined based on conditions which is especially useful in scenarios where the same configuration needs to adapt to different platforms or compilers. Jinja2 is a dependency of Conan and is installed automatically alongside it.

You can verify the templating engine is installed with:
```bash
pip show jinja2
```

If Jinja2 is installed, this will display package information: name, version, and installation path.

##### Usage Example

The templating engine can be used effectively in Conan profiles to create universal configurations that automatically determine parameter values based on the characteristics of the system where Conan is being executed — for example:
```ini
[settings]
os={{ detect_api.detect_os() }}
arch={{ detect_api.detect_arch() }}
build_type=Debug
compiler=clang
compiler.version=14
compiler.libcxx=libstdc++11
compiler.cppstd=gnu20

[conf]
tools.build:compiler_executables={"c":"clang","cpp":"clang++"}
tools.build:jobs={{ os.cpu_count() }}
tools.system.package_manager:mode=install
```

This template uses Conan's special `detect_api` object and the standard Python `os` module to automatically detect characteristics such as the operating system, system architecture, and the number of available logical CPU cores on the machine where Conan is running.

More usage examples for the templating engine can be found in the [official documentation](https://docs.conan.io/2/reference/config_files/profiles.html#profile-rendering).