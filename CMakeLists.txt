cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")

HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.15.6.tar.gz"
    SHA1 "3dc0cd2ccbc52e8be93967dbc583303a1e825aa2"
)

project(crashpad)

add_subdirectory(client)
