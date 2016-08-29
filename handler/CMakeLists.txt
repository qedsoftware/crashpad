set(
    sources
    crash_report_upload_thread.cc
    crash_report_upload_thread.h
    handler_main.cc
    handler_main.h
    prune_crash_reports_thread.cc
    prune_crash_reports_thread.h
)

if(APPLE)
  list(
      APPEND
      sources
      mac/crash_report_exception_handler.cc
      mac/crash_report_exception_handler.h
      mac/exception_handler_server.cc
      mac/exception_handler_server.h
  )
endif()

if(WIN32)
  list(
      APPEND
      sources
      win/crash_report_exception_handler.cc
      win/crash_report_exception_handler.h
  )
endif()

add_library(crashpad_handler_lib ${sources})

target_include_directories(
    crashpad_handler_lib
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
)

hunter_add_package(mini_chromium)
find_package(mini_chromium CONFIG REQUIRED)

target_link_libraries(
    crashpad_handler_lib
    PUBLIC
    mini_chromium::base
    crashpad_compat
    crashpad_util
    crashpad_snapshot
    crashpad_minidump
    crashpad_client
    crashpad_tool_support
)

add_executable(crashpad_handler main.cc)
target_link_libraries(crashpad_handler PUBLIC crashpad_handler_lib)

install(
    TARGETS crashpad_handler
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

if(WIN32 AND CRASHPAD_BUILD_TESTS)
  add_executable(crashy_program win/crashy_test_program.cc)
  target_link_libraries(
      crashy_program
      PUBLIC
      mini_chromium::base
      crashpad_client
  )

  add_executable(crash_other_program win/crash_other_program.cc)
  target_link_libraries(
      crash_other_program
      PUBLIC
      mini_chromium::base
      crashpad_client
      crashpad_util
      crashpad_test
  )

  add_executable(hanging_program win/hanging_program.cc)
  target_link_libraries(
      hanging_program
      PUBLIC
      mini_chromium::base
      crashpad_client
  )

  add_library(loader_lock_dll MODULE win/loader_lock_dll.cc)

  add_executable(self_destroying_program win/self_destroying_test_program.cc)
  target_link_libraries(
      self_destroying_program
      PUBLIC
      mini_chromium::base
      crashpad_client
      crashpad_snapshot
  )

  if(WIN_X86_ARCH)
    add_executable(crashy_z7_loader win/crashy_test_z7_loader.cc)
    target_link_libraries(
        crashy_z7_loader
        PUBLIC
        mini_chromium::base
        crashpad_client
        crashpad_test
    )
  endif()
endif()
