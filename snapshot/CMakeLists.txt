set(
    sources
    capture_memory.cc
    capture_memory.h
    cpu_architecture.h
    cpu_context.cc
    cpu_context.h
    crashpad_info_client_options.cc
    crashpad_info_client_options.h
    exception_snapshot.h
    handle_snapshot.cc
    handle_snapshot.h
    memory_snapshot.h
    minidump/minidump_simple_string_dictionary_reader.cc
    minidump/minidump_simple_string_dictionary_reader.h
    minidump/minidump_string_list_reader.cc
    minidump/minidump_string_list_reader.h
    minidump/minidump_string_reader.cc
    minidump/minidump_string_reader.h
    minidump/module_snapshot_minidump.cc
    minidump/module_snapshot_minidump.h
    minidump/process_snapshot_minidump.cc
    minidump/process_snapshot_minidump.h
    module_snapshot.h
    process_snapshot.h
    system_snapshot.h
    thread_snapshot.h
    unloaded_module_snapshot.cc
    unloaded_module_snapshot.h
)

if(APPLE)
  list(
      APPEND
      sources
      mac/cpu_context_mac.cc
      mac/cpu_context_mac.h
      mac/exception_snapshot_mac.cc
      mac/exception_snapshot_mac.h
      mac/mach_o_image_annotations_reader.cc
      mac/mach_o_image_annotations_reader.h
      mac/mach_o_image_reader.cc
      mac/mach_o_image_reader.h
      mac/mach_o_image_segment_reader.cc
      mac/mach_o_image_segment_reader.h
      mac/mach_o_image_symbol_table_reader.cc
      mac/mach_o_image_symbol_table_reader.h
      mac/memory_snapshot_mac.cc
      mac/memory_snapshot_mac.h
      mac/module_snapshot_mac.cc
      mac/module_snapshot_mac.h
      mac/process_reader.cc
      mac/process_reader.h
      mac/process_snapshot_mac.cc
      mac/process_snapshot_mac.h
      mac/process_types.cc
      mac/process_types.h
      mac/process_types/all.proctype
      mac/process_types/crashpad_info.proctype
      mac/process_types/crashreporterclient.proctype
      mac/process_types/custom.cc
      mac/process_types/dyld_images.proctype
      mac/process_types/flavors.h
      mac/process_types/internal.h
      mac/process_types/loader.proctype
      mac/process_types/nlist.proctype
      mac/process_types/traits.h
      mac/system_snapshot_mac.cc
      mac/system_snapshot_mac.h
      mac/thread_snapshot_mac.cc
      mac/thread_snapshot_mac.h
  )
endif()

if(WIN32)
  list(
      APPEND
      sources
      win/cpu_context_win.cc
      win/cpu_context_win.h
      win/exception_snapshot_win.cc
      win/exception_snapshot_win.h
      win/capture_memory_delegate_win.cc
      win/capture_memory_delegate_win.h
      win/memory_map_region_snapshot_win.cc
      win/memory_map_region_snapshot_win.h
      win/memory_snapshot_win.cc
      win/memory_snapshot_win.h
      win/module_snapshot_win.cc
      win/module_snapshot_win.h
      win/pe_image_annotations_reader.cc
      win/pe_image_annotations_reader.h
      win/pe_image_reader.cc
      win/pe_image_reader.h
      win/pe_image_resource_reader.cc
      win/pe_image_resource_reader.h
      win/process_reader_win.cc
      win/process_reader_win.h
      win/process_snapshot_win.cc
      win/process_snapshot_win.h
      win/process_subrange_reader.cc
      win/process_subrange_reader.h
      win/system_snapshot_win.cc
      win/system_snapshot_win.h
      win/thread_snapshot_win.cc
      win/thread_snapshot_win.h
  )
endif()

add_library(crashpad_snapshot ${sources})

target_include_directories(
    crashpad_snapshot
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
)

if(WIN32)
  add_library(
      crashpad_snapshot_api
      api/module_annotations_win.cc
      api/module_annotations_win.h
  )

  target_include_directories(
      crashpad_snapshot_api
      PUBLIC
      "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
  )

  target_link_libraries(
      crashpad_snapshot_api PUBLIC crashpad_compat crashpad_util
  )
endif()

hunter_add_package(mini_chromium)
find_package(mini_chromium CONFIG REQUIRED)

target_link_libraries(
    crashpad_snapshot PUBLIC mini_chromium::base crashpad_compat crashpad_util
)

if(WIN32)
  target_link_libraries(crashpad_snapshot PUBLIC "powrprof") # unresolved external symbol _CallNtPowerInformation@20
endif()

if(CRASHPAD_BUILD_TESTS)
  add_library(
      crashpad_snapshot_test_lib
      test/test_cpu_context.cc
      test/test_cpu_context.h
      test/test_exception_snapshot.cc
      test/test_exception_snapshot.h
      test/test_memory_map_region_snapshot.cc
      test/test_memory_map_region_snapshot.h
      test/test_memory_snapshot.cc
      test/test_memory_snapshot.h
      test/test_module_snapshot.cc
      test/test_module_snapshot.h
      test/test_process_snapshot.cc
      test/test_process_snapshot.h
      test/test_system_snapshot.cc
      test/test_system_snapshot.h
      test/test_thread_snapshot.cc
      test/test_thread_snapshot.h
  )
  target_link_libraries(
      crashpad_snapshot_test_lib
      PUBLIC
      crashpad_snapshot
      crashpad_compat
      mini_chromium::base
      crashpad_util
  )

  set(
      sources
      cpu_context_test.cc
      crashpad_info_client_options_test.cc
      minidump/process_snapshot_minidump_test.cc
  )
  if(WIN32)
    list(
        APPEND
        sources
        api/module_annotations_win_test.cc
        win/cpu_context_win_test.cc
        win/exception_snapshot_win_test.cc
        win/extra_memory_ranges_test.cc
        win/pe_image_annotations_reader_test.cc
        win/pe_image_reader_test.cc
        win/process_reader_win_test.cc
        win/process_snapshot_win_test.cc
        win/system_snapshot_win_test.cc
    )
  endif()
  if(APPLE)
    list(
        APPEND
        sources
        mac/cpu_context_mac_test.cc
        mac/mach_o_image_annotations_reader_test.cc
        mac/mach_o_image_reader_test.cc
        mac/mach_o_image_segment_reader_test.cc
        mac/process_reader_test.cc
        mac/process_types_test.cc
        mac/system_snapshot_mac_test.cc
    )
  endif()

  hunter_add_package(GTest)
  find_package(GTest CONFIG REQUIRED)

  add_executable(crashpad_snapshot_test ${sources})

  target_link_libraries(
      crashpad_snapshot_test
      PUBLIC
      crashpad_snapshot
      crashpad_client
      crashpad_compat
      crashpad_test
      GTest::main
      mini_chromium::base
      crashpad_util
  )
  if(WIN32)
    target_link_libraries(
        crashpad_snapshot_test
        PUBLIC
        crashpad_snapshot_api
    )
  endif()

  if(APPLE)
    target_link_libraries(
        crashpad_snapshot_test
        PUBLIC
        "-framework OpenCL"
    )
  endif()

  if(WIN32)
    target_link_libraries(
        crashpad_snapshot_test
        PUBLIC
        crashpad_snapshot_test_image_reader_module
    )
  endif()

  add_custom_command(
      TARGET
      crashpad_snapshot_test
      POST_BUILD
      COMMAND
      "${CMAKE_COMMAND}"
      -E
      copy
      "$<TARGET_FILE:crashpad_snapshot_test_module_crashy_initializer>"
      "$<TARGET_FILE_DIR:crashpad_snapshot_test>"
  )

  add_test(
      NAME crashpad_snapshot_test
      COMMAND crashpad_snapshot_test --gtest_filter=-SystemSnapshotMacTest.TimeZone # FIXME
      WORKING_DIRECTORY "$<TARGET_FILE_DIR:crashpad_snapshot_test>"
  )

  add_library(
      crashpad_snapshot_test_module MODULE crashpad_info_client_options_test_module.cc
  )

  set_target_properties(
      crashpad_snapshot_test_module
      PROPERTIES
      "DEBUG_POSTFIX" ""
      PREFIX ""
  )

  target_link_libraries(
      crashpad_snapshot_test_module
      PUBLIC
      mini_chromium::base
      crashpad_client
  )

  if(APPLE)
    add_library(
        crashpad_snapshot_test_module_crashy_initializer
        MODULE
        mac/mach_o_image_annotations_reader_test_module_crashy_initializer.cc
    )
    set_target_properties(
        crashpad_snapshot_test_module_crashy_initializer
        PROPERTIES
        "DEBUG_POSTFIX" ""
        PREFIX ""
    )

    add_executable(
        crashpad_snapshot_test_no_op
        mac/mach_o_image_annotations_reader_test_no_op.cc
    )
    add_test(
        NAME crashpad_snapshot_test_no_op
        COMMAND crashpad_snapshot_test_no_op
    )
  endif()

  if(WIN32)
    add_executable(
        crashpad_snapshot_test_crashing_child
        win/crashpad_snapshot_test_crashing_child.cc
    )
    target_link_libraries(
        crashpad_snapshot_test_crashing_child
        PUBLIC
        crashpad_client
        crashpad_compat
        mini_chromium::base
        crashpad_util
    )
    add_test(
        NAME crashpad_snapshot_test_crashing_child
        COMMAND crashpad_snapshot_test_crashing_child
    )

    add_executable(
        crashpad_snapshot_test_dump_without_crashing
        win/crashpad_snapshot_test_dump_without_crashing.cc
    )
    target_link_libraries(
        crashpad_snapshot_test_dump_without_crashing
        PUBLIC
        crashpad_client
        crashpad_compat
        mini_chromium::base
        crashpad_util
    )
    add_test(
        NAME crashpad_snapshot_test_dump_without_crashing
        COMMAND crashpad_snapshot_test_dump_without_crashing
    )

    add_executable(
        crashpad_snapshot_test_extra_memory_ranges
        win/crashpad_snapshot_test_extra_memory_ranges.cc
    )
    target_link_libraries(
        crashpad_snapshot_test_extra_memory_ranges
        PUBLIC
        crashpad_client
        crashpad_compat
        mini_chromium::base
    )
    add_test(
        NAME crashpad_snapshot_test_extra_memory_ranges
        COMMAND crashpad_snapshot_test_extra_memory_ranges
    )

    add_executable(
        crashpad_snapshot_test_image_reader
        win/crashpad_snapshot_test_image_reader.cc
    )
    target_link_libraries(
        crashpad_snapshot_test_image_reader
        PUBLIC
        crashpad_client
        crashpad_compat
        mini_chromium::base
        crashpad_util
    )
    add_test(
        NAME crashpad_snapshot_test_image_reader
        COMMAND crashpad_snapshot_test_image_reader
    )

    add_library(
        crashpad_snapshot_test_image_reader_module
        win/crashpad_snapshot_test_image_reader_module.cc
    )
    target_link_libraries(
        crashpad_snapshot_test_image_reader_module
        PUBLIC
        crashpad_client
        mini_chromium::base
    )

    add_executable(
        crashpad_snapshot_test_simple_annotations
        win/crashpad_snapshot_test_simple_annotations.cc
    )
    target_link_libraries(
        crashpad_snapshot_test_simple_annotations
        PUBLIC
        crashpad_client
        crashpad_compat
        mini_chromium::base
    )
    add_test(
        NAME crashpad_snapshot_test_simple_annotations
        COMMAND crashpad_snapshot_test_simple_annotations
    )
  endif()
endif()
