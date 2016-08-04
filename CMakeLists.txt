cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")

HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.15.6.tar.gz"
    SHA1 "3dc0cd2ccbc52e8be93967dbc583303a1e825aa2"
)

project(crashpad)

if(APPLE)
  add_subdirectory(third_party/apple_cctools)
endif()

if(WIN32)
  add_subdirectory(third_party/getopt)
endif()

add_subdirectory(compat)
add_subdirectory(util)

add_subdirectory(client)
