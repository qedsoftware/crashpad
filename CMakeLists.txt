cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")

HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.15.11.tar.gz"
    SHA1 "c9abc02add6c4b45b15e6cac1ed926432590ee38"
)

project(crashpad VERSION 0.0.1)

if(WIN32)
  enable_language(ASM_MASM)
else()
  enable_language(ASM)
endif()

option(CRASHPAD_BUILD_TESTS "Build tests" ON)

if(CRASHPAD_BUILD_TESTS)
  enable_testing()
endif()

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-exceptions")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti")
else()
  STRING (REGEX REPLACE "/RTC(su|[1su])" "" CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")
endif()

####
# Installation (https://github.com/forexample/package-example)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

# Include module with fuction 'write_basic_package_version_file'
include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use variables:
#   * targets_export_name
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    DIRECTORY client compat handler minidump snapshot util
    DESTINATION "${include_install_dir}"
    FILES_MATCHING PATTERN "*.h"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

###

if(APPLE)
  add_subdirectory(third_party/apple_cctools)
endif()

if(WIN32)
  add_subdirectory(third_party/getopt)
endif()

add_subdirectory(compat)
add_subdirectory(util)

add_subdirectory(client)
add_subdirectory(snapshot)
add_subdirectory(minidump)
add_subdirectory(tools)
if(CRASHPAD_BUILD_TESTS)
  add_subdirectory(test)
endif()
add_subdirectory(handler)
