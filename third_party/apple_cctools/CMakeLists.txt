add_library(
    apple_cctools
    cctools/include/mach-o/getsect.h
    cctools/libmacho/getsecbyname.c
)

target_include_directories(
    apple_cctools PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/../..>"
)
