# Effective CMake
[C++Now 2017 Daniel Pfeifer](https://www.youtube.com/watch?v=bsXLMQ6WgIk&feature=youtu.be&t=37m15s)
[slide](https://www.slideshare.net/DanielPfeifer1/effective-cmake)

## CMake is code
use the same principles for `CMakeLists.txt` and modules as for the rest of your codebase

## Language

### Organization
- **Directories** that contain a `CMakeLists.txt` are the entry point for the build system generator. Subdirectories may be added with `add_subdirectory()` and must contain a `CMakeLists.txt` too.
- **Scripts** are `<script>.cmake` files that can be executed with `cmake -P <script>.cmake`. Not all commands are supported
- **Modules** are `<script>.cmake` files located in the `CMAKE_MODULE_PATH`. Modules can be loaded with the `include()` command

### Commands
`command_name(space separated list of strings)`
- Scripting commands change state of command processor
  + set variables
  + change behavior of other commands
- Project commands
  + create build targets
  + modify build targets
- Command invocations are not expressions

### Variables
```
set(hello world)
message(STATUS "hello, ${hello}")
```
- Set with the `set()` command
- Expand with `${}`
- Variables and values are strings
- Lists are `;` separated strings
- CMake variables are not environment variables (unlike `Makefile`)
- Unset variable expands to empty string

### Comments
```
# a single line comment

#[==[
  multi line comments

  #[=[
    may be nested
  ]=]
]==]
```

### Generator expressions
```
target_compile_definitions(foo PRIVATE
  "VERBOSITY=$<IF:$<CONFIG:Debug>,30,10>"
  )
```
- Generator expressions use the `$<>` syntax
- Not evaluated by command interpreter.
    It is just a string with `$<>`
- Evaluated during build system generation
- Not supported in all commands (obviously)

## Custom Commands

### Two types of commands
- Commands can be added with `function()` or `macro()`
- Difference is like in `C++`
- When a new command replaces an existing command, the old one can be accessed with a `_` prefix

### Custom command: Function
```
# definition
function(my_command input output)
  # ...
  set(${output} ... PARENT_SCOPE)
endfunction()
# use
my_command(foo bar)
```
- Variables are scoped to the function, unless set with `PARENT_SCOPE`
- Available variables: `input, output, ARGC, ARGV, ARGN, ARG0, ARG1, ARG2, ...`
- Example: `${output}` expands to `bar`

### Custom command: Macro
```
macro(my_command input output)
  # ...
endmacro()
my_command(foo bar)
```
- No extra scope
- Text replacements: `${input}`, `${output}`, `${ARGC}`, `${ARGV}`, `${ARGN}`, `${ARG0}`, `${ARG1}`, `${ARG2}`, ...
- Example: `${output}` is replaced by bar

### Create macros to wrap commands that have output parameters. Otherwise, create a function

## Evolving CMake code

## Targets and Properties
***Variables are so CMake 2.8.13. Moden CMake is about Targets and Properties***

### Look Ma, no Variables!
```
add_library(Foo foo.cpp)
target_link_libraries(Foo PRIVATE Bar::Bar)

if (WIN32)
  target_sources(Foo PRIVATE foo_win32.cpp)
  target_link_libraries(Foo PRIVATE Bar::Win32Support)
endif()
```
***Avoid Custom variables in the arguments of project commands***

***Don't use file(GLOB) in projects***

### Imagine Targets as Objects
- Constructors:
  + add_executable()
  + add_library()
- Member varibles:
  + Target properties (too many to list here)
- Member functions:
  + get_target_property()
  + set_target_properties()
  + get_property(TARGET)
  + set_property(TARGET)
  + target_compile_definitions()
  + target_compile_features()
  + target_compile_options()
  + target_include_directories()
  + target_link_libraries()
  + target_sources()

### Forget those commands:
  add_compile_options()
  include_directories()
  link_directories()
  link_libraries()

### Example:
```
target_compile_features(Foo
  PUBLIC
    cxx_strong_enums
  PRIVATE
    cxx_lambdas
    cxx_range_for
  )
```
- Add `cxx_strong_enums` to the target properties `COMPILE_FEATUERS` and `INTERFACE_COMPILE_FEATURES`
- Add `cxx_lambdas;cxx_range_for` to the target property `COMPILE_FEATURES`

***Get your hands off CMAKE_CXX_FLAGS***

### Build Specification and Usage Requirements
- Non-`INTERFACE_` properties define the *build specifictation* of a target
- `INTERFACE_` properties define the *usage requirements* of a target
- `PRIVATE` populates the non-`INTERFACE_` property
- `PUBLIC` populates *both*

***Use `target_link_libraries()` to express *direct* dependencies***

### Example:
```
target_link_libraries(Foo
  PUBLIC Bar::Bar
  PRIVATE Cow::Cow
  )
```
- Adds `Bar::Bar` to the target properties `LINK_LIBRARIES` and `INTERFACE_LINK_LIBRARIES`
- Adds `Cow::Cow` to the target property `LINK_LIBRARIES`
- Effectively adds all `INTERFACE_<property>` of `Bar::Bar` to `<property>` and `INTERFACE_<property>`
- Effectively adds all `INTERFACE_<property>` of `Cow::Cow` to `<property>`
- Adds `$<LINK_ONLY:Cow::Cow>` to `INTERFACE_LINK_LIBRARIES`

### Pure usage requirements
```
add_library(Bar INTERFACE)
target_compile_definitions(Bar INTERFACE BAR=1)
```
- `INTERFACE` libraries have no *build specifictation*
- They only have *usage requirements*

***Don't abuse requirements! Eg: -Wall is not a requirement!***

## Project Boundaries

### How to use external libraries
Always like this:
```
find_package(Foo 2.0 REQUIRED)
# ...
target_link_libraries(... Foo::Foo ...)
```

### FindFoo.cmake
```
find_path(Foo_INCLUDE_DIR foo.h)
find_library(Foo_LIBRARY foo)
mark_as_advanced(Foo_INCLUDE_DIR Foo_LIBRARY)

include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(Foo REQUIRED_VARS Foo_LIBRARY Foo_INCLUDE_DIR
  )

if(Foo_FOUND and NOT TARGET Foo::Foo)
  add_library(Foo::Foo UNKNOWN IMPORTED)
  set_target_properties(Foo::Foo PROPERTIES
    IMPORTED_LINK_INTERFACE_LANGUAGES "CXX"
    IMPORTED_LOCATION "${Foo_LIBRARY}"
    INTERFACE_INCLUDE_DIRECTORIES "${Foo_INCLUDE_DIR}"
    )
endif()
```

### Export your library interface!
```
find_package(Bar 2.0 REQUIRED)
add_library(Foo ...)
target_link_libraries(Foo PRIVATE Bar::Bar)

install(TARGETS Foo EXPORT FooTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION lib
  INCLUDES DESTINATION lib
  )
install(EXPORT FooTargets
  FILE FooTargets.cmake
  NAMESPACE Foo::
  DESTINATION lib/cmake/Foo
  )
include(CMakePackageConfigureHelpers)
write_basic_package_version_file("FooConfigVersion.cmake"
  VERSION ${Foo_VERSION}
  COMPATIBILITY SameMajorVersion
  )
install(FILES "FooConfig.cmake" "FooConfigVersion.cmake"
  DESTINATION lib/cmake/Foo
  )
```

```
include(CMakeFindDependencyMacro)
find_dependency(Bar 2.0)
include("${CMAKE_CURRENT_LIST_DIR}/FooTargets.cmake")
```

### Export the right information!
*Warning:*
The library interface may change during installation. Use the `BUILD_INTERFACE` and `INSTALL_INTERFACE` generator expressions as filters
```
target_include_directories(Foo PUBLIC
  $<BUILD_INTERFACE:${Foo_BINARY_DIR}/include>
  $<BUILD_INTERFACE:${Foo_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  )
```

## Creating Packages

### CPack
- *CPack* is packaging tool distributed with *CMake*
- `set()` variables in `CPackConfig.cmake`, or
- `set()` variables in `CMakeLists.txt` and `include(CPack)`

***Write your own CPackageConfig.cmake and include() the one that is generated by CMake***

### CPack secret
The variable `CPACK_INSTALL_CMAKE_PROJECTS` is a list of quadruples:
1. Build directory
2. Project Name
3. Project Compoent
4. Directory

### Packaging multiple configurations
1. Make sure different configurations don't collide:
`set(CMAKE_DEBUG_POSTFIX "-d")`
2. Create separate build directories for debug, release
3. Use this *CPackConfig.cmake*
```
include("release/CPackConfig.cmake")
set(CPACK_INSTALL_CMAKE_PROJECTS
  "debug;Foo;ALL;/"
  "release;Foo;ALL;/"
  )
```

## Package Management

### my requirements for a package manager
- support system packages
- support prebuild libraries
- support building dependencies as subprojects
- do not require any change to my projects!

### How to use external libraries
Always like this:
```
find_package(Foo 2.0 REQUIRED)
# ...
target_link_libraries(... Foo::Foo ...)
```

### Do not require any changes to my projects!
- System packages ...
  + work out of the box
- Prebuilt libraries ...
  + need to be put into `CMAKE_PREFIX_PATH`
- Subprojects
  + We need to turn find_package(Foo) into a no-op
  + What about the imported target `Foo::Foo`?

### Use the your public interface
when you export `Foo` in namespace `Foo::`, also create an alias `Foo::Foo`
`add_library(Foo::Foo ALIAS Foo)`

### The toplevel super-project
```
set(CMAKE_PREFIX_PATH "/prefix")
set(as_subproject Foo)

macro(find_package)
  if(NOT "${ARG0}" IN_LIST as_subproject)
    _find_package(${ARGV})
  endif()
endmacro()

add_subdirectory(Foo)
add_subdirectory(App)
```

### How does that work?
If `Foo` is a ...
- system package:
  + `find_package(Foo)` either finds `FooConfig.cmake` in the system or uses `FindFoo.cmake` to find the library in the system.
  + in either case, the target `Foo::Foo` is imported
- prebuild library:
  + `find_package(Foo)` either finds `FooConfig.cmake` in the `CMAKE_PREFIX_PATH` or uses `FindFoo.cmake` to find the library in the `CMAKE_PREFIX_PATH`.
  + in either case, the target `Foo::Foo` is imported
- subproject:
  + `find_package(Foo)` does nothing.
  + The target `Foo::Foo` is part of project

## CTest

### Run with `ctest -S build.cmake`
```
set(CTEST_SOURCE_DIRECTORY "/source")
set(CTEST_BINARY_DIRECTORY "/binary")

set(ENV{CXXFLAGS} "--coverage")
set(CTEST_CMAKE_GENERATOR "Ninja")
set(CTEST_USE_LAUNCHERS 1)

set(CTEST_COVERAGE_COMMAND "gcov")
set(CTEST_MEMORYCHECK_COMMAND "valgrind")
# set(CTEST_MEMORYCHECK_TYPE "ThreadSanitizer")

ctest_start("Continuous")
ctest_configure()
ctest_build()
ctest_test()
ctest_coverage()
ctest_memcheck()
ctest_submit()
```

***CTest scripts are the right place for CI specific settings. Keep that information out out the project.***

### Filtering tests by name


## Cross Compiling

### Toolchain.cmake

***Don't put logic in toolchain files.***

## Static Analysis
