add_library(
    apple_cctools
    cctools/include/mach-o/getsect.h
    cctools/libmacho/getsecbyname.c
)

target_include_directories(
    apple_cctools PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/../..>"
)

install(
    TARGETS apple_cctools
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)
