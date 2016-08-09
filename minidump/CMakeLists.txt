set(
    sources
    minidump_context.h
    minidump_context_writer.cc
    minidump_context_writer.h
    minidump_crashpad_info_writer.cc
    minidump_crashpad_info_writer.h
    minidump_exception_writer.cc
    minidump_exception_writer.h
    minidump_extensions.cc
    minidump_extensions.h
    minidump_file_writer.cc
    minidump_file_writer.h
    minidump_handle_writer.cc
    minidump_handle_writer.h
    minidump_memory_info_writer.cc
    minidump_memory_info_writer.h
    minidump_memory_writer.cc
    minidump_memory_writer.h
    minidump_misc_info_writer.cc
    minidump_misc_info_writer.h
    minidump_module_crashpad_info_writer.cc
    minidump_module_crashpad_info_writer.h
    minidump_module_writer.cc
    minidump_module_writer.h
    minidump_rva_list_writer.cc
    minidump_rva_list_writer.h
    minidump_simple_string_dictionary_writer.cc
    minidump_simple_string_dictionary_writer.h
    minidump_stream_writer.cc
    minidump_stream_writer.h
    minidump_string_writer.cc
    minidump_string_writer.h
    minidump_system_info_writer.cc
    minidump_system_info_writer.h
    minidump_thread_id_map.cc
    minidump_thread_id_map.h
    minidump_thread_writer.cc
    minidump_thread_writer.h
    minidump_unloaded_module_writer.cc
    minidump_unloaded_module_writer.h
    minidump_user_stream_writer.cc
    minidump_user_stream_writer.h
    minidump_writable.cc
    minidump_writable.h
    minidump_writer_util.cc
    minidump_writer_util.h
)

add_library(crashpad_minidump ${sources})

target_include_directories(
    crashpad_minidump
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
)

hunter_add_package(mini_chromium)
find_package(mini_chromium CONFIG REQUIRED)

target_link_libraries(
    crashpad_minidump
    PUBLIC
    mini_chromium::base
    crashpad_compat
    crashpad_snapshot
)

if(CRASHPAD_BUILD_TESTS)
  hunter_add_package(GTest)
  find_package(GTest CONFIG REQUIRED)

  add_executable(
      crashpad_minidump_test
      minidump_context_writer_test.cc
      minidump_crashpad_info_writer_test.cc
      minidump_exception_writer_test.cc
      minidump_handle_writer_test.cc
      minidump_file_writer_test.cc
      minidump_memory_info_writer_test.cc
      minidump_memory_writer_test.cc
      minidump_misc_info_writer_test.cc
      minidump_module_crashpad_info_writer_test.cc
      minidump_module_writer_test.cc
      minidump_rva_list_writer_test.cc
      minidump_simple_string_dictionary_writer_test.cc
      minidump_string_writer_test.cc
      minidump_system_info_writer_test.cc
      minidump_thread_id_map_test.cc
      minidump_thread_writer_test.cc
      minidump_unloaded_module_writer_test.cc
      minidump_user_stream_writer_test.cc
      minidump_writable_test.cc
      test/minidump_context_test_util.cc
      test/minidump_context_test_util.h
      test/minidump_file_writer_test_util.cc
      test/minidump_file_writer_test_util.h
      test/minidump_memory_writer_test_util.cc
      test/minidump_memory_writer_test_util.h
      test/minidump_rva_list_test_util.cc
      test/minidump_rva_list_test_util.h
      test/minidump_string_writer_test_util.cc
      test/minidump_string_writer_test_util.h
      test/minidump_writable_test_util.cc
      test/minidump_writable_test_util.h
  )

  target_link_libraries(
      crashpad_minidump_test
      PUBLIC
      crashpad_minidump
      crashpad_snapshot_test_lib
      crashpad_test
      GTest::main
      mini_chromium::base
      crashpad_util
  )

  add_test(NAME crashpad_minidump_test COMMAND crashpad_minidump_test)
endif()
