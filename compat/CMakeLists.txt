set(
    sources
    non_cxx11_lib/type_traits
    non_cxx11_lib/utility
)

if(APPLE)
  list(
      APPEND
      sources
      mac/AvailabilityMacros.h
      mac/kern/exc_resource.h
      mac/mach/mach.h
      mac/mach-o/getsect.cc
      mac/mach-o/getsect.h
      mac/mach-o/loader.h
      mac/sys/resource.h
  )
else()
  list(
      APPEND
      sources
      non_mac/mach/mach.h
  )
endif()

if(WIN32)
  list(
      APPEND
      sources
      win/getopt.h
      win/strings.cc
      win/strings.h
      win/sys/types.h
      win/time.cc
      win/time.h
      win/winnt.h
  )
else()
  list(
      APPEND
      sources
      non_win/dbghelp.h
      non_win/minwinbase.h
      non_win/timezoneapi.h
      non_win/verrsrc.h
      non_win/windows.h
      non_win/winnt.h
  )
endif()

if(APPLE OR WIN32)
  add_library(crashpad_compat ${sources})
else()
  # No real sources left for the target
  message(FATAL_ERROR "Platform ${CMAKE_SYSTEM_NAME} not supported")
endif()

if(APPLE)
  target_link_libraries(crashpad_compat PUBLIC apple_cctools)
  target_include_directories(
      crashpad_compat
      PUBLIC
      "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/mac>"
      "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/non_cxx11_lib>"
  )
endif()

if(WIN32)
  target_link_libraries(crashpad_compat PUBLIC getopt)
  target_include_directories(
      crashpad_compat
      PUBLIC
      "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/win>"
  )
else()
  target_include_directories(
      crashpad_compat
      PUBLIC
      "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/non_win>"
  )
endif()

install(
    TARGETS crashpad_compat
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)
