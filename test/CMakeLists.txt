set(
    sources
    errors.cc
    errors.h
    file.cc
    file.h
    gtest_death_check.h
    multiprocess.h
    multiprocess_exec.h
    paths.cc
    paths.h
    scoped_temp_dir.cc
    scoped_temp_dir.h
)

if(APPLE)
  list(
      APPEND
      sources
      mac/dyld.h
      mac/mach_errors.cc
      mac/mach_errors.h
      mac/mach_multiprocess.cc
      mac/mach_multiprocess.h
      paths_mac.cc
  )
endif()

if(WIN32)
  list(
      APPEND
      sources
      win/child_launcher.cc
      win/child_launcher.h
      win/win_child_process.cc
      win/win_child_process.h
      win/win_multiprocess.cc
      win/win_multiprocess.h
      multiprocess_exec_win.cc
      paths_win.cc
      scoped_temp_dir_win.cc
  )
else()
  list(
      APPEND
      sources
      multiprocess_exec_posix.cc
      multiprocess_posix.cc
      scoped_temp_dir_posix.cc
  )
endif()

add_library(crashpad_test ${sources})
target_include_directories(
    crashpad_test PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
)

hunter_add_package(mini_chromium)
find_package(mini_chromium CONFIG REQUIRED)

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)

target_link_libraries(
    crashpad_test
    PUBLIC
    crashpad_compat
    mini_chromium::base
    crashpad_util
    GTest::gtest
)
