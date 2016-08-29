add_library(
    crashpad_tool_support tool_support.cc tool_support.h
)

target_include_directories(
    crashpad_tool_support
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
)

hunter_add_package(mini_chromium)
find_package(mini_chromium CONFIG REQUIRED)

target_link_libraries(crashpad_tool_support PUBLIC mini_chromium::base)

add_executable(crashpad_database_util crashpad_database_util.cc)
target_link_libraries(
    crashpad_database_util PUBLIC crashpad_tool_support crashpad_client
)

add_executable(generate_dump generate_dump.cc)
target_link_libraries(
    generate_dump PUBLIC crashpad_tool_support crashpad_minidump crashpad_util
)

install(
    TARGETS generate_dump crashpad_database_util
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

if(APPLE)
  set(
      plist_flags
      "-sectcreate __TEXT __info_plist ${CMAKE_CURRENT_LIST_DIR}/mac/sectaskaccess_info.plist"
  )

  target_link_libraries(generate_dump PUBLIC "${plist_flags}")

  add_executable(catch_exception_tool mac/catch_exception_tool.cc)
  target_link_libraries(
      catch_exception_tool
      PUBLIC
      crashpad_tool_support
      crashpad_util
  )

  add_executable(exception_port_tool mac/exception_port_tool.cc)
  target_link_libraries(
      exception_port_tool
      PUBLIC
      crashpad_tool_support
      "${plist_flags}"
  )
  target_link_libraries(
      exception_port_tool PUBLIC crashpad_util
  )

  add_executable(on_demand_service_tool mac/on_demand_service_tool.mm)
  target_link_libraries(
      on_demand_service_tool PUBLIC crashpad_tool_support crashpad_util
  )

  add_executable(run_with_crashpad mac/run_with_crashpad.cc)
  target_link_libraries(
      run_with_crashpad
      PUBLIC
      crashpad_tool_support
      crashpad_util
      crashpad_client
  )

  install(
      TARGETS
      catch_exception_tool
      exception_port_tool
      on_demand_service_tool
      run_with_crashpad
      EXPORT "${targets_export_name}"
      LIBRARY DESTINATION "lib"
      ARCHIVE DESTINATION "lib"
      RUNTIME DESTINATION "bin"
      INCLUDES DESTINATION "${include_install_dir}"
  )
endif()
