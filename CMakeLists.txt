cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")

HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.15.7.tar.gz"
    SHA1 "ebf3121da17d5f73fddfd1ca88da104bc599581d"
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
