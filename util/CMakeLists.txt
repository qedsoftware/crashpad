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
