set(
    sources
    crash_report_database.cc
    crash_report_database.h
    crashpad_client.h
    crashpad_info.cc
    crashpad_info.h
    prune_crash_reports.cc
    prune_crash_reports.h
    settings.cc
    settings.h
    simple_string_dictionary.cc
    simple_string_dictionary.h
    simple_address_range_bag.cc
    simple_address_range_bag.h
    simulate_crash.h
)

if(APPLE)
  list(
      APPEND
      sources
      capture_context_mac.S
      capture_context_mac.h
      crash_report_database_mac.mm
      crashpad_client_mac.cc
      simulate_crash_mac.cc
      simulate_crash_mac.h
  )
endif()

if(WIN32)
  list(
      APPEND
      sources
      crash_report_database_win.cc
      crashpad_client_win.cc
      simulate_crash_win.h
  )
endif()

add_library(crashpad_client ${sources})
target_include_directories(
    crashpad_client PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
)

hunter_add_package(mini_chromium)
find_package(mini_chromium CONFIG REQUIRED)

target_link_libraries(
    crashpad_client PUBLIC mini_chromium::base crashpad_util crashpad_compat
)

install(
    TARGETS crashpad_client
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

if(CRASHPAD_BUILD_TESTS)
  hunter_add_package(GTest)
  find_package(GTest CONFIG REQUIRED)
  find_package(GMock CONFIG REQUIRED)

  set(
      sources
      crash_report_database_test.cc
      prune_crash_reports_test.cc
      settings_test.cc
      simple_address_range_bag_test.cc
      simple_string_dictionary_test.cc
  )

  if(APPLE)
    list(APPEND sources capture_context_mac_test.cc simulate_crash_mac_test.cc)
  endif()

  if(WIN32)
    list(APPEND sources crashpad_client_win_test.cc)
  endif()

  add_executable(crashpad_client_test ${sources})

  target_link_libraries(
      crashpad_client_test
      PUBLIC
      crashpad_client
      crashpad_handler_lib
      crashpad_test
      GMock::main
      GTest::gtest
      mini_chromium::base
      crashpad_util
  )

  add_test(
      NAME crashpad_client_test COMMAND crashpad_client_test
  )
endif()
