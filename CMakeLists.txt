cmake_minimum_required(VERSION 3.5)
project(drlibs VERSION 0.10.0)

#########################################################################################
# DR WAV
#########################################################################################
option(DR_WAV "Compile and export dr_wav.h" ON)

# DR WAV target
if(DR_WAV)
    add_library(dr_wav STATIC src/dr_wav.c)
    target_include_directories(dr_wav PUBLIC
        "$<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>")
    target_compile_definitions(dr_wav PUBLIC DR_WAV)

    install(TARGETS dr_wav EXPORT drlibsTargets)
    install(FILES include/drlibs/dr_wav.h DESTINATION include/drlibs)

endif(DR_WAV)

#########################################################################################
# Package Configuration
#########################################################################################
# Helpers functions for creating config files that can be included by other projects
include(CMakePackageConfigHelpers)

# Dummy target for generating export drlibsTargets even if no drlibs target was enabled
add_library(none INTERFACE)
install(TARGETS none EXPORT drlibsTargets)

# Create config package
configure_package_config_file(
    "Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/drlibsConfig.cmake"
    INSTALL_DESTINATION "lib/cmake/drlibs")
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/drlibsConfigVersion.cmake"
    COMPATIBILITY SameMajorVersion)

# Install config package
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/drlibsConfig.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/drlibsConfigVersion.cmake"
        DESTINATION "lib/cmake/drlibs")
install(EXPORT "drlibsTargets"
        NAMESPACE "drlibs::"
        DESTINATION "lib/cmake/drlibs")
