
# ODB as ORM for C++ Backend Project

ODB is an open-source, cross-platform, and cross-database object-relational mapping (ORM) system for C++. It allows you to persist C++ objects in a relational database without working directly with tables, columns, or SQL and without writing mapping code by hand.

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->
   * [Why ODB?](#why-odb)
   * [Integrate into Project](#integrate-into-project)
      + [With deb Package](#with-deb-package)
      + [With Conan Package Manager](#with-conan-package-manager)
   * [Describing Models](#describing-models)
   * [Generating Support Files](#generating-support-files)
   * [Database Connector](#database-connector)
   * [Usage](#usage)
   * [References](#references)
<!-- TOC end -->

## Why ODB?

It is better described in the [to-dos-api-cpp](https://github.com/TourmalineCore/to-dos-api-cpp/blob/master/docs/technologies.md#orm) repository.

## Integrate into Project

### With deb Package

> When downloading a pre-built package, note that it is only available for Debian-based systems on the x86-64 architecture.

Downloads are available [here](https://www.codesynthesis.com/download/odb/2.5.0/).

Pick the base package [libodb_2.5.0-0~ubuntu22.04_amd64.deb](https://www.codesynthesis.com/download/odb/2.5.0/ubuntu/ubuntu22.04/x86_64/libodb_2.5.0-0~ubuntu22.04_amd64.deb) along with its dev version [libodb-dev_2.5.0-0~ubuntu22.04_amd64.deb](https://www.codesynthesis.com/download/odb/2.5.0/ubuntu/ubuntu22.04/x86_64/libodb-dev_2.5.0-0~ubuntu22.04_amd64.deb).

Then pick the necessary database driver — for example, for PostgreSQL: [libodb-pgsql_2.5.0-0~ubuntu22.04_amd64.deb](https://www.codesynthesis.com/download/odb/2.5.0/ubuntu/ubuntu22.04/x86_64/libodb-pgsql_2.5.0-0~ubuntu22.04_amd64.deb) along with its dev version [libodb-pgsql-dev_2.5.0-0~ubuntu22.04_amd64.deb](https://www.codesynthesis.com/download/odb/2.5.0/ubuntu/ubuntu22.04/x86_64/libodb-pgsql-dev_2.5.0-0~ubuntu22.04_amd64.deb).

Then install them with `dpkg`:

```bash
dpkg -i libodb_2.5.0-0~debian12_amd64.deb && \
dpkg -i ibodb-dev_2.5.0-0~ubuntu22.04_amd64.deb && \
dpkg -i libodb-pgsql_2.5.0-0~ubuntu22.04_amd64.deb && \
dpkg -i libodb-pgsql-dev_2.5.0-0~ubuntu22.04_amd64.deb
```

To link the packages into your project you may use any build system (CMake, Meson, etc.). As an example, we will use CMake.

Add the following directives to your project's `CMakeLists.txt`:

```cmake
target_link_libraries(${YOUR_PROJECT_TARGET} PUBLIC 
  odb odb-pgsql
)
```

### With Conan Package Manager

> We are currently preparing a recipe for [conan-center-index](https://github.com/conan-io/conan-center-index) for automatic builds. Once the recipe is successfully added, this option will be much simpler.

> Installing Conan into your project is described [here](backend/package-manager-conan.md).

First, you need to create a Conan `local-recipes-index`. To do that, create a `deps` folder in your project's root directory and place the following structure inside it:

```bash
.
└── recipes
    ├── libodb
    │   ├── all
    │   │   └── conanfile.py
    │   └── config.yml
    └── libodb-pgsql
        ├── all
        │   └── conanfile.py
        └── config.yml
```

Place the following content in each `config.yml`:

```yml
versions:
  "2.5.0": # version of libodb*
    folder: all
```

For the `conanfile.py` of `libodb`:

<details>
<summary>Content of deps/recipes/libodb/all/conanfile.py</summary>

```python
import os
import textwrap
from conan import ConanFile
from conan.tools.cmake import CMake, CMakeToolchain, cmake_layout
from conan.tools.files import get, save, rmdir



class LibOdbConan(ConanFile):
    name = "libodb"
    version = "2.5.0"
    description = "ODB C++ ORM — core runtime library"
    license = "GPL-2.0-only"
    homepage = "https://www.codesynthesis.com/products/odb/"
    topics = ("odb", "orm", "database", "c++")


    settings = "os", "compiler", "build_type", "arch"
    options  = {"shared": [True, False], "fPIC": [True, False]}
    default_options = {"shared": False, "fPIC": True}


    def configure(self):
        if self.options.shared:
            self.options.rm_safe("fPIC")


    def layout(self):
        cmake_layout(self, src_folder="src")


    def validate(self):
        if self.settings.os != "Linux":
            raise ConanInvalidConfiguration(
                f"{self.ref} is supported only on Linux"
            )
        try:
            with open("/etc/os-release") as f:
                info = dict(
                    line.strip().split("=", 1)
                    for line in f if "=" in line
                )
            distro  = info.get("ID", "").strip('"')
            version = info.get("VERSION_ID", "").strip('"')
            if distro != "ubuntu" or version != "22.04":
                raise ConanInvalidConfiguration(
                    f"{self.ref} is supported only on Ubuntu 22.04 "
                    f"(detected: {distro} {version})"
                )
        except FileNotFoundError:
            raise ConanInvalidConfiguration(
                f"{self.ref} requires Ubuntu 22.04 (/etc/os-release not found)"
            )


    def source(self):
        get(
            self,
            url=f"https://www.codesynthesis.com/download/odb/{self.version}/libodb-{self.version}.tar.gz",
            sha256="700038a73c6cbead011129b15030b7cdd3f73510b687f2c4504808df4230441b",  # replace hash if you changing sources
            strip_root=True,
        )
        self._inject_cmake()


    def _inject_cmake(self):
        major = self.version.split(".")[0]
        cmake = textwrap.dedent(f"""\
            cmake_minimum_required(VERSION 3.15)
            project(libodb VERSION {self.version} LANGUAGES CXX)


            file(GLOB          ODB_SOURCES         "odb/*.cxx")
            file(GLOB_RECURSE  ODB_DETAILS_SOURCES "odb/details/*.cxx")


            add_library(odb
                ${{ODB_SOURCES}}
                ${{ODB_DETAILS_SOURCES}}
            )


            target_include_directories(odb PUBLIC
                $<BUILD_INTERFACE:${{CMAKE_CURRENT_SOURCE_DIR}}>
                $<INSTALL_INTERFACE:include>
            )


            target_compile_features(odb PUBLIC cxx_std_11)


            find_package(Threads REQUIRED)
            target_compile_definitions(odb PUBLIC ODB_THREADS_POSIX)
            target_link_libraries(odb PUBLIC Threads::Threads)


            set_target_properties(odb PROPERTIES
                VERSION   {self.version}
                SOVERSION {major}
            )


            install(TARGETS odb
                ARCHIVE DESTINATION lib
                LIBRARY DESTINATION lib
                RUNTIME DESTINATION bin
            )
            install(DIRECTORY odb/
                DESTINATION include/odb
                FILES_MATCHING
                    PATTERN "*.hxx"
                    PATTERN "*.ixx"
                    PATTERN "*.txx"
                    PATTERN "*.h"
            )
        """)
        save(self, os.path.join(self.source_folder, "CMakeLists.txt"), cmake)


    def generate(self):
        tc = CMakeToolchain(self)
        tc.generate()


    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()


    def package(self):
        cmake = CMake(self)
        cmake.install()
        rmdir(self, os.path.join(self.package_folder, "lib", "pkgconfig"))
        rmdir(self, os.path.join(self.package_folder, "share"))


    def package_info(self):
        self.cpp_info.libs = ["odb"]
        self.cpp_info.system_libs = ["pthread"]
        self.cpp_info.set_property("cmake_target_name", "libodb::libodb")
        self.cpp_info.set_property("pkg_config_name",   "libodb")
```
</details>

For the `conanfile.py` of `libodb-pgsql`:

<details>
<summary>Content of deps/recipes/libodb-pgsql/all/conanfile.py</summary>

```python
import os
import textwrap
from conan import ConanFile
from conan.errors import ConanInvalidConfiguration
from conan.tools.cmake import CMake, CMakeToolchain, CMakeDeps, cmake_layout
from conan.tools.files import get, save, rmdir


class LibOdbPgsqlConan(ConanFile):
    name = "libodb-pgsql"
    version = "2.5.0"
    license = "GPL-2.0-only"
    homepage = "https://www.codesynthesis.com/products/odb/"
    topics = ("odb", "orm", "database", "c++")


    settings = "os", "compiler", "build_type", "arch"
    options  = {"shared": [True, False], "fPIC": [True, False]}
    default_options = {"shared": False, "fPIC": True}


    def config_options(self):
        pass


    def configure(self):
        if self.options.shared:
            self.options.rm_safe("fPIC")


    def layout(self):
        cmake_layout(self, src_folder="src")


    def validate(self):
        if self.settings.os != "Linux":
            raise ConanInvalidConfiguration(
                f"{self.ref} is supported only on Linux"
            )
        try:
            with open("/etc/os-release") as f:
                info = dict(
                    line.strip().split("=", 1)
                    for line in f if "=" in line
                )
            distro  = info.get("ID", "").strip('"')
            version = info.get("VERSION_ID", "").strip('"')
            if distro != "ubuntu" or version != "22.04":
                raise ConanInvalidConfiguration(
                    f"{self.ref} is supported only on Ubuntu 22.04 "
                    f"(detected: {distro} {version})"
                )
        except FileNotFoundError:
            raise ConanInvalidConfiguration(
                f"{self.ref} requires Ubuntu 22.04 (/etc/os-release not found)"
            )


    def requirements(self):
        self.requires("libodb/2.5.0")
        self.requires("libpq/17.7")


    def source(self):
        get(
            self,
            url=f"https://www.codesynthesis.com/download/odb/{self.version}/libodb-pgsql-{self.version}.tar.gz",
            sha256="f6e63db4a2f77604f48115f64c74a5854ca20f03f208568966693e95712a3e17", # replace hash if you changing sources
            strip_root=True,
        )
        self._inject_cmake()


    def _inject_cmake(self):
        major = self.version.split(".")[0]
        cmake = textwrap.dedent(f"""\
            cmake_minimum_required(VERSION 3.15)
            project(odb-pgsql VERSION {self.version} LANGUAGES CXX)


            find_package(libodb     REQUIRED CONFIG)
            find_package(PostgreSQL REQUIRED CONFIG)


            file(GLOB ODB_SOURCES "odb/pgsql/*.cxx")


            file(GLOB ODB_PREGENERATED_SOURCES
                "odb/pgsql/details/pregenerated/odb/pgsql/details/*.cxx"
            )


            add_library(odb-pgsql
                ${{ODB_SOURCES}}
                ${{ODB_PREGENERATED_SOURCES}}
            )


            target_include_directories(odb-pgsql PUBLIC
                $<BUILD_INTERFACE:${{CMAKE_CURRENT_SOURCE_DIR}}>
                $<BUILD_INTERFACE:${{CMAKE_CURRENT_SOURCE_DIR}}/odb/pgsql/details/pregenerated>
                $<INSTALL_INTERFACE:include>
            )


            target_compile_features(odb-pgsql PUBLIC cxx_std_11)


            target_link_libraries(odb-pgsql PUBLIC
                libodb::libodb
                PostgreSQL::PostgreSQL
            )


            set_target_properties(odb-pgsql PROPERTIES
                VERSION   {self.version}
                SOVERSION {major}
                POSITION_INDEPENDENT_CODE ON
            )


            install(TARGETS odb-pgsql
                ARCHIVE DESTINATION lib
                LIBRARY DESTINATION lib
                RUNTIME DESTINATION bin
            )


            install(DIRECTORY odb/pgsql
                DESTINATION include/odb
                FILES_MATCHING
                    PATTERN "*.hxx"
                    PATTERN "*.ixx"
                    PATTERN "*.txx"
                    PATTERN "*.h"
                PATTERN "pregenerated" EXCLUDE
            )


            install(DIRECTORY odb/pgsql/details/pregenerated/odb/pgsql/details
                DESTINATION include/odb/pgsql
                FILES_MATCHING
                    PATTERN "*.hxx"
                    PATTERN "*.ixx"
            )
        """)
        save(self, os.path.join(self.source_folder, "CMakeLists.txt"), cmake)


    def generate(self):
        tc = CMakeToolchain(self)
        tc.generate()


        deps = CMakeDeps(self)
        deps.generate()


    def build(self):
        cmake = CMake(self)
        cmake.configure()
        cmake.build()


    def package(self):
        cmake = CMake(self)
        cmake.install()
        rmdir(self, os.path.join(self.package_folder, "lib", "pkgconfig"))
        rmdir(self, os.path.join(self.package_folder, "share"))


    def package_info(self):
        self.cpp_info.libs = ["odb-pgsql"]
        self.cpp_info.requires = [
            "libodb::libodb",
            "libpq::pq",
        ]
        self.cpp_info.set_property("cmake_target_name", "libodb-pgsql::libodb-pgsql")
        self.cpp_info.set_property("pkg_config_name",   "libodb-pgsql")
```
</details>

After filling in the recipe `conanfile.py` files, register the local recipes index with the following command:

```bash
conan remote add local-recipes ./deps --type=local-recipes-index
```

Then add the packages to your project's `conanfile.py`:

```python
...
def requirements(self):
    ...
    self.requires("libodb/2.5.0")
    self.requires("libodb-pgsql/2.5.0")
...
```

Finally, add the following directives to your project's `CMakeLists.txt`:

```cmake
...

find_package(libodb REQUIRED)
find_package(libodb-pgsql REQUIRED)

target_link_libraries(${YOUR_PROJECT_TARGET} PUBLIC 
  ...
  libodb::libodb
  libodb-pgsql::libodb-pgsql
)
...
```

## Describing Models

Models are described as regular C++ classes annotated with `#pragma db` directives. ODB uses these pragmas as instructions for code generation.

A minimal model example:

<details>
<summary>Example of a to-do model</summary>

```cpp
// to-do.h
#pragma once


#include <ctime>
#include <odb/core.hxx>
#include <odb/nullable.hxx>
#include <string>


#pragma db object table("todo")
class ToDo
{
    public:
    ToDo() = default;
    ToDo(const std::string& name, std::time_t createdAtUtc)
    : name_(name),
      createdAtUtc_(createdAtUtc),
      deletedAtUtc_()
    {}


    std::uint64_t id() const { return id_; }
    const std::string& name() const { return name_; }
    std::time_t createdAtUtc() const { return createdAtUtc_; }
    const odb::nullable<std::time_t>& deletedAtUtc() const { return deletedAtUtc_; }


    void name(const std::string& n) { name_ = n; }
    void createdAtUtc(std::time_t t) { createdAtUtc_ = t; }
    void deletedAtUtc(std::time_t t) { deletedAtUtc_ = t; }


private:
    friend class odb::access;


#pragma db id auto
    std::uint64_t id_;


    std::string name_;
#pragma db type("BIGINT")
    std::time_t createdAtUtc_;


#pragma db null
#pragma db type("BIGINT")
    odb::nullable<std::time_t> deletedAtUtc_;
};
```
</details>

More about models describing is writed in official [documentation](https://www.codesynthesis.com/products/odb/doc/manual.xhtml#3.2).

## Generating Support Files

Generating support files is a required step when working with ODB. First, download the ODB CLI tool from [odb_2.5.0-0~ubuntu22.04_amd64.deb](https://www.codesynthesis.com/download/odb/2.5.0/ubuntu/ubuntu22.04/x86_64/odb_2.5.0-0~ubuntu22.04_amd64.deb) and install it with `dpkg -i odb_2.5.0-0~ubuntu22.04_amd64.deb`

To generate support files for all models at once, use the following script:

```bash
ROOT_DIR="$(pwd)" && \
MODELS_DIR="$ROOT_DIR/path/to/your/models" && \
ODB_OUT_DIR="$MODELS_DIR/odb-gen" && \
find "$MODELS_DIR" -type f \( -name "*.hxx" -o -name "*.h" \) \
    -not -path "$ODB_OUT_DIR/*" -print0 | \
while IFS= read -r -d '' header; do
    echo "  → $(basename "$header")"
    odb --std c++20 -d pgsql --generate-query \
        -o "$ODB_OUT_DIR" \
        -I "$ROOT_DIR" \
        "$header"
done
```

<details>
<summary>Variant for the Conan installation of ODB</summary>

```bash
ROOT_DIR="$(pwd)" && \
MODELS_DIR="$ROOT_DIR/path/to/your/models" && \
ODB_OUT_DIR="$MODELS_DIR/odb-gen" && \
ODB_INCLUDE="$(conan cache path libodb/2.5.0)/../s/src" && \
PGSQL_INCLUDE="$(conan cache path libodb-pgsql/2.5.0)/../s/src" && \
find "$MODELS_DIR" -type f \( -name "*.hxx" -o -name "*.h" \) \
    -not -path "$ODB_OUT_DIR/*" -print0 | \
while IFS= read -r -d '' header; do
    echo "  → $(basename "$header")"
    odb --std c++20 -d pgsql --generate-query \
        -o "$ODB_OUT_DIR" \
        -I "$ODB_INCLUDE" \
        -I "$PGSQL_INCLUDE" \
        -I "$ROOT_DIR" \
        "$header"
done
```
</details>

More about ODB CLI described in official [documentation](https://www.codesynthesis.com/products/odb/doc/odb.xhtml).

## Database Connector

Here is a quick way to create a database connection class:

```cpp
// db_connector.h
#pragma once


#include <memory>
#include <odb/database.hxx>


class DbConnection
{
public:
    static std::shared_ptr<odb::database> get();


private:
    DbConnection() = default;
    static std::shared_ptr<odb::database> instance_;
};
```

```cpp
// db_connector.cpp
#include "db_connection.h"
#include "../utils/app-config/app-config.h"


#include <cstdlib>
#include <stdexcept>


#include <odb/pgsql/database.hxx>


std::shared_ptr<odb::database> DbConnection::instance_ = nullptr;


std::shared_ptr<odb::database> DbConnection::get()
{
    static std::once_flag flag;
    std::call_once(
        flag,
        []()
        {
            auto& config = AppConfig::GetInstance();


            const std::string host = "127.0.0.1";   // Host of database
            const std::string port = "5432";        // Port of database;
            const std::string user = "db_user";     // User of database;
            const std::string password = "passwd";  // Users password
            const std::string dbname = "default";   // Database name


            std::string conn = "host=" + host + " port=" + port + " dbname=" + dbname + " user=" + user + " password=" + password;


            instance_ = std::shared_ptr<odb::database>(new odb::pgsql::database(conn)); // Create DbConnection singletone instance
        }
    );


    return instance_;
}
```

## Usage

You can then easily use DbConnection class instance in commands and queries as shown in the following example:

```cpp
// to-do-query.h
#pragma once


#include "data/models/to-do.h"
#include <memory>
#include <odb/database.hxx>
#include <vector>


class ToDoQueries
{
public:
    ToDoQueries(odb::database& db)
    : db_(db)
    {}
    std::shared_ptr<ToDo> get_todo_by_id(int id);


private:
    odb::database& db_;
};
```

```cpp
// to-do-query.cpp
#include "to-do-query.h"
#include "data/models/odb-gen/to-do-odb.hxx"
#include <memory>
#include <odb/transaction.hxx>
std::shared_ptr<std::vector<ToDo>> ToDoQueries::get_all_todos()
{
    odb::transaction t(db_.begin()); // Open transaction on database


    odb::result<ToDo> r = db_.query<ToDo>(odb::query<ToDo>()); // Requesting of all to-dos from database


    auto todos = std::make_shared<std::vector<ToDo>>(); //
    for (auto i = r.begin(); i != r.end(); ++i)         // Parse inner ODB structure on vector
        todos->push_back(*i);                           //


    t.commit(); // Close the transaction
    return todos;
}
```

For more details on working with ODB commands and queries, refer to the official documentation:

- [Making objects persistent and other write operations](https://www.codesynthesis.com/products/odb/doc/manual.xhtml#3.8)
- [Querying the Database](https://www.codesynthesis.com/products/odb/doc/manual.xhtml#4)

***

## References

- [to-dos-api-cpp](https://github.com/TourmalineCore/to-dos-api-cpp)
- [Creating a Conan file](https://docs.conan.io/2/reference/conanfile.html)
- [Local Recipes Index Repository](https://docs.conan.io/2/devops/devops_local_recipes_index.html)
- [ODB Manual](https://www.codesynthesis.com/products/odb/doc/manual.xhtml)