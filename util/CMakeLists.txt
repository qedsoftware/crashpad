set(
    sources
    file/file_io.cc
    file/file_io.h
    file/file_reader.cc
    file/file_reader.h
    file/file_seeker.cc
    file/file_seeker.h
    file/file_writer.cc
    file/file_writer.h
    file/string_file.cc
    file/string_file.h
    misc/clock.h
    misc/implicit_cast.h
    misc/initialization_state.h
    misc/initialization_state_dcheck.cc
    misc/initialization_state_dcheck.h
    misc/pdb_structures.cc
    misc/pdb_structures.h
    misc/random_string.cc
    misc/random_string.h
    misc/scoped_forbid_return.cc
    misc/scoped_forbid_return.h
    misc/symbolic_constants_common.h
    misc/tri_state.h
    misc/uuid.cc
    misc/uuid.h
    net/http_body.cc
    net/http_body.h
    net/http_headers.cc
    net/http_headers.h
    net/http_multipart_builder.cc
    net/http_multipart_builder.h
    net/http_transport.cc
    net/http_transport.h
    numeric/checked_address_range.cc
    numeric/checked_address_range.h
    numeric/checked_range.h
    numeric/in_range_cast.h
    numeric/int128.h
    numeric/safe_assignment.h
    stdlib/aligned_allocator.cc
    stdlib/aligned_allocator.h
    stdlib/cxx.h
    stdlib/map_insert.h
    stdlib/objc.h
    stdlib/pointer_container.h
    stdlib/string_number_conversion.cc
    stdlib/string_number_conversion.h
    stdlib/strlcpy.cc
    stdlib/strlcpy.h
    stdlib/strnlen.cc
    stdlib/strnlen.h
    string/split_string.cc
    string/split_string.h
    synchronization/semaphore.h
    thread/thread.cc
    thread/thread.h
    thread/thread_log_messages.cc
    thread/thread_log_messages.h
    thread/worker_thread.cc
    thread/worker_thread.h
)

if(APPLE)
  list(
      APPEND
      sources
      mac/checked_mach_address_range.h
      mac/launchd.h
      mac/launchd.mm
      mac/mac_util.cc
      mac/mac_util.h
      mac/service_management.cc
      mac/service_management.h
      mac/xattr.cc
      mac/xattr.h
      mach/child_port_handshake.cc
      mach/child_port_handshake.h
      mach/child_port_server.cc
      mach/child_port_server.h
      mach/child_port_types.h
      mach/composite_mach_message_server.cc
      mach/composite_mach_message_server.h
      mach/exc_client_variants.cc
      mach/exc_client_variants.h
      mach/exc_server_variants.cc
      mach/exc_server_variants.h
      mach/exception_behaviors.cc
      mach/exception_behaviors.h
      mach/exception_ports.cc
      mach/exception_ports.h
      mach/exception_types.cc
      mach/exception_types.h
      mach/mach_extensions.cc
      mach/mach_extensions.h
      mach/mach_message.cc
      mach/mach_message.h
      mach/mach_message_server.cc
      mach/mach_message_server.h
      mach/notify_server.cc
      mach/notify_server.h
      mach/scoped_task_suspend.cc
      mach/scoped_task_suspend.h
      mach/symbolic_constants_mach.cc
      mach/symbolic_constants_mach.h
      mach/task_for_pid.cc
      mach/task_for_pid.h
      mach/task_memory.cc
      mach/task_memory.h
      misc/clock_mac.cc
      net/http_transport_mac.mm
      posix/process_info_mac.cc
      synchronization/semaphore_mac.cc
  )
endif()

if(WIN32)
  list(
      APPEND
      sources
      file/file_io_win.cc
      misc/clock_win.cc
      net/http_transport_win.cc
      synchronization/semaphore_win.cc
      thread/thread_win.cc
      win/address_types.h
      win/capture_context.asm
      win/capture_context.h
      win/checked_win_address_range.h
      win/command_line.cc
      win/command_line.h
      win/critical_section_with_debug_info.cc
      win/critical_section_with_debug_info.h
      win/exception_handler_server.cc
      win/exception_handler_server.h
      win/get_function.cc
      win/get_function.h
      win/get_module_information.cc
      win/get_module_information.h
      win/handle.cc
      win/handle.h
      win/module_version.cc
      win/module_version.h
      win/nt_internals.cc
      win/nt_internals.h
      win/ntstatus_logging.cc
      win/ntstatus_logging.h
      win/process_info.cc
      win/process_info.h
      win/process_structs.h
      win/registration_protocol_win.cc
      win/registration_protocol_win.h
      win/scoped_handle.cc
      win/scoped_handle.h
      win/scoped_local_alloc.cc
      win/scoped_local_alloc.h
      win/scoped_process_suspend.cc
      win/scoped_process_suspend.h
      win/time.cc
      win/time.h
      win/xp_compat.h
  )

  # TODO: Why COMPILE_OPTIONS not working?
  get_source_file_property(
      options win/capture_context.asm COMPILE_FLAGS
  )
  if(options)
    message(FATAL_ERROR "Unexpected")
  endif()
  set_source_files_properties(
      win/capture_context.asm
      PROPERTIES COMPILE_FLAGS "/safeseh"
  )
else()
  list(
      APPEND
      sources
      file/file_io_posix.cc
      misc/clock_posix.cc
      posix/close_multiple.cc
      posix/close_multiple.h
      posix/close_stdio.cc
      posix/close_stdio.h
      posix/drop_privileges.cc
      posix/drop_privileges.h
      posix/process_info.h
      posix/symbolic_constants_posix.cc
      posix/symbolic_constants_posix.h
      synchronization/semaphore_posix.cc
      thread/thread_posix.cc
  )
endif()

if(APPLE)
  set(def_relative_files "exc.defs" "mach_exc.defs" "notify.defs")
  set(input_files "")
  foreach(x ${def_relative_files})
    # CMAKE_OSX_SYSROOT may be empty (e.g. for Makefile generators),
    # in this case files will be taken from root.
    set(full_path "${CMAKE_OSX_SYSROOT}/usr/include/mach/${x}")
    if(NOT EXISTS "${full_path}")
      message(FATAL_ERROR "File not found: ${full_path}")
    endif()
    list(APPEND input_files "${full_path}")
  endforeach()
  list(APPEND input_files "${CMAKE_CURRENT_LIST_DIR}/mach/child_port.defs")

  find_package(PythonInterp 2.7 REQUIRED)

  set(output_dir "${CMAKE_CURRENT_BINARY_DIR}/util/mach")
  file(MAKE_DIRECTORY "${output_dir}")

  # Create generate rule for each input file. Add each generated output
  # as a source to the target.
  foreach(input ${input_files})
    get_filename_component(name_we "${input}" NAME_WE)
    set(output_files "")
    foreach(suffix "User.c" "Server.c" ".h" "Server.h")
      list(APPEND output_files "${output_dir}/${name_we}${suffix}")
    endforeach()
    add_custom_command(
        OUTPUT
        ${output_files}
        COMMAND
        "${PYTHON_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/mach/mig.py" "${input}" ${output_files}
        DEPENDS
        "${CMAKE_CURRENT_SOURCE_DIR}/mach/mig.py" "${input}"
    )
    list(APPEND sources ${output_files})
  endforeach()
endif()

add_library(crashpad_util ${sources})

hunter_add_package(mini_chromium)
find_package(mini_chromium CONFIG REQUIRED)

target_link_libraries(crashpad_util PUBLIC mini_chromium::base crashpad_compat)
target_include_directories(
    crashpad_util PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/..>"
)

if(APPLE)
  target_include_directories(
      crashpad_util PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>"
  )
  target_link_libraries(crashpad_util PUBLIC "-framework IOKit")
  target_link_libraries(crashpad_util PUBLIC bsm)
endif()

if(WIN32)
  target_link_libraries(crashpad_util PUBLIC "rpcrt4") # unresolved external symbol __imp__UuidCreate@4
  target_link_libraries(crashpad_util PUBLIC "version") # unresolved external symbol _GetFileVersionInfoSizeW@8
  target_link_libraries(crashpad_util PUBLIC "winhttp") # unresolved external symbol __imp__WinHttpQueryHeaders@24
endif()

install(
    TARGETS crashpad_util
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

if(CRASHPAD_BUILD_TESTS)
  if(WIN32)
    add_executable(
        crashpad_util_test_process_info_test_child
        win/process_info_test_child.cc
    )

    target_compile_definitions(
        crashpad_util_test_process_info_test_child PUBLIC "UNICODE" "_UNICODE"
    )

    if(MSVC)
      set_target_properties(
          crashpad_util_test_process_info_test_child
          PROPERTIES
          LINK_FLAGS
          /BASE:0x78000000
          /DYNAMICBASE:NO
          /FIXED
      )
    endif()
  endif()

  set(
      sources
      file/file_io_test.cc
      file/string_file_test.cc
      misc/clock_test.cc
      misc/initialization_state_dcheck_test.cc
      misc/initialization_state_test.cc
      misc/scoped_forbid_return_test.cc
      misc/random_string_test.cc
      misc/uuid_test.cc
      net/http_body_test.cc
      net/http_body_test_util.cc
      net/http_body_test_util.h
      net/http_multipart_builder_test.cc
      net/http_transport_test.cc
      numeric/checked_address_range_test.cc
      numeric/checked_range_test.cc
      numeric/in_range_cast_test.cc
      numeric/int128_test.cc
      stdlib/aligned_allocator_test.cc
      stdlib/map_insert_test.cc
      stdlib/string_number_conversion_test.cc
      stdlib/strlcpy_test.cc
      stdlib/strnlen_test.cc
      string/split_string_test.cc
      synchronization/semaphore_test.cc
      thread/thread_log_messages_test.cc
      thread/thread_test.cc
      thread/worker_thread_test.cc
  )

  if(APPLE)
    list(
        APPEND
        sources
        mac/launchd_test.mm
        mac/mac_util_test.mm
        mac/service_management_test.mm
        mac/xattr_test.cc
        mach/child_port_handshake_test.cc
        mach/child_port_server_test.cc
        mach/composite_mach_message_server_test.cc
        mach/exc_client_variants_test.cc
        mach/exc_server_variants_test.cc
        mach/exception_behaviors_test.cc
        mach/exception_ports_test.cc
        mach/exception_types_test.cc
        mach/mach_extensions_test.cc
        mach/mach_message_server_test.cc
        mach/mach_message_test.cc
        mach/notify_server_test.cc
        mach/scoped_task_suspend_test.cc
        mach/symbolic_constants_mach_test.cc
        mach/task_memory_test.cc
    )
  endif()

  if(WIN32)
    list(
        APPEND
        sources
        win/capture_context_test.cc
        win/command_line_test.cc
        win/critical_section_with_debug_info_test.cc
        win/exception_handler_server_test.cc
        win/get_function_test.cc
        win/handle_test.cc
        win/process_info_test.cc
        win/scoped_process_suspend_test.cc
        win/time_test.cc
    )
  else()
    list(
        APPEND
        sources
        posix/process_info_test.cc
        posix/symbolic_constants_posix_test.cc
    )
  endif()

  hunter_add_package(GTest)
  find_package(GTest CONFIG REQUIRED)
  find_package(GMock CONFIG REQUIRED)

  add_executable(crashpad_util_test ${sources})

  target_link_libraries(
      crashpad_util_test
      PUBLIC
      crashpad_util
      crashpad_client
      crashpad_compat
      crashpad_test
      GMock::main
      GTest::gtest
      mini_chromium::base
  )

  set(dst_dir "$<TARGET_FILE_DIR:crashpad_util_test>/util/net/")
  set(src_txt "${PROJECT_SOURCE_DIR}/util/net/testdata/ascii_http_body.txt")
  set(src_dat "${PROJECT_SOURCE_DIR}/util/net/testdata/binary_http_body.dat")
  set(src_py "${PROJECT_SOURCE_DIR}/util/net/http_transport_test_server.py")
  if(NOT EXISTS "${src_txt}")
    message(FATAL_ERROR "File not found: ${src_txt}")
  endif()
  if(NOT EXISTS "${src_dat}")
    message(FATAL_ERROR "File not found: ${src_dat}")
  endif()
  if(NOT EXISTS "${src_py}")
    message(FATAL_ERROR "File not found: ${src_py}")
  endif()

  add_custom_command(
      TARGET crashpad_util_test
      POST_BUILD
      COMMAND
      "${CMAKE_COMMAND}" -E make_directory "${dst_dir}/testdata"
      COMMAND
      "${CMAKE_COMMAND}" -E copy "${src_txt}" "${dst_dir}/testdata"
      COMMAND
      "${CMAKE_COMMAND}" -E copy "${src_dat}" "${dst_dir}/testdata"
      COMMAND
      "${CMAKE_COMMAND}" -E copy "${src_py}" "${dst_dir}"
  )

  if(WIN32)
    target_link_libraries(crashpad_util_test PUBLIC "imagehlp") # unresolved external symbol __imp__GetTimestampForLoadedLibrary@4
  endif()

  add_test(
      NAME crashpad_util_test
      COMMAND crashpad_util_test --gtest_filter=-ProcessInfo.OtherProcessWOW64 # FIXME
      WORKING_DIRECTORY "$<TARGET_FILE_DIR:crashpad_util_test>"
  )
endif()
