# drlibs-cmake-package

CMake config package for the [dr\_libs](https://github.com/mackron/dr_libs) single file audio decoding libraries for C/C++.

Supports only `dr_wav.h` for now. Other headers can be added later.

## Usage

This projects wraps the dr\_libs headers in static libraries and exports the targets in a CMake Config Package,
so that other projects can use `find_package(drlibs)` for including the drlibs libraries into their targets.

See the options in the [CMakeLists.txt](CMakeLists.txt) for configuring the build.   
The enabled options are passed on to the exported targets as public compile definitions, so the target linking to
the dr\_libs component will also have this defines in the target's properties.

### Example 1:

#### Building and installing on the system

```sh
mkdir build
cd build
cmake ..  # you can define here an install path with -DCMAKE_INSTALL_PREFIX=<path>
cmake --build .
cmake --install .
```

#### Including in your project

```cmake
cmake_minimum_required(VERSION 3.5)
project(Example)

find_package(drlibs 0.10.0 REQUIRED COMPONENTS dr_wav)

add_executable(example main.c)
target_link_libraries(example PRIVATE drlibs::dr_wav)
```

### Alternative 2: Integrate into your project build step

One hacky way to always build the drlibs libraries along with your project, is adding them as cmake `ExternalProject` dependencies:

```cmake
cmake_minimum_required(VERSION 3.5)
project(SuperProject)
include(ExternalProject)

set(EXTERNAL_LIBS_INSTALL_DIR ${CMAKE_CURRENT_BINARY_DIR}/external)

# Add this drlibs directory as an external project
ExternalProject_Add(drlibs_external
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/drlibs
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/drlibs
    INSTALL_DIR ${EXTERNAL_LIBS_INSTALL_DIR}
    CMAKE_ARGS
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        -DCMAKE_GENERATOR=${CMAKE_GENERATOR}
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DDR_WAV=ON)

# Also include your project sources directory as a sub-project
ExternalProject_Add(SubProject
    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/SubProject
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/SubProject
    BUILD_ALWAYS 1
    CMAKE_ARGS
        -DCMAKE_GENERATOR=${CMAKE_GENERATOR}
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_PREFIX_PATH="${EXTERNAL_LIBS_INSTALL_DIR};${CMAKE_PREFIX_PATH}"
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        -DBUILD_SHARED_LIBS=${BUILD_SHARED_LIBS}
    DEPENDS drlibs_external)

# Note the DEPENDS and CMAKE_PREFIX_PATH parameters,
# this will make CMake build the drlibs_external project before the SubProject,
# for that the drlibs project artifacts (lib/include and config package) will be available
# for the SubProject's `find_package`s at ${CMAKE_CURRENT_BINARY_DIR}/external.
```

Then your CMakeLists.txt in the source code directory can look just like the one above in *Including in your project*.

## LICENSE

All source code LICENSE rights to https://github.com/mackron/dr\_libs.
