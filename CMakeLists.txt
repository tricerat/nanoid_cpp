cmake_minimum_required(VERSION 3.5)
project(nanoid_cpp
  VERSION 2.1.0
)
set(CMAKE_CXX_STANDARD 11)
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

add_subdirectory(vendor/catch2)

file(GLOB_RECURSE NANOID_SRC_FILES src/*.cpp)
file(GLOB_RECURSE NANOID_INC_FILES inc/*.h)

function(create_project SUBPROJECT_TARGET)
  add_library(${SUBPROJECT_TARGET} ${NANOID_SRC_FILES})
  target_include_directories(${SUBPROJECT_TARGET} PUBLIC inc)
  target_link_libraries(${SUBPROJECT_TARGET} Threads::Threads)

  # target_compile_options(nanoid PRIVATE
  # $<$<CXX_COMPILER_ID:MSVC>:/W4>
  # $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -pedantic>
  # )
  set_target_properties(${SUBPROJECT_TARGET} PROPERTIES FOLDER "third-party")
  set_property(TARGET ${SUBPROJECT_TARGET} PROPERTY POSITION_INDEPENDENT_CODE ON)

  # ###############################################################################
  # Compile and link options
  # ###############################################################################
  if(MSVC)
    if(ARGV1 STREQUAL "static")
      set_target_properties(${SUBPROJECT_TARGET} PROPERTIES
        MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
      )
    else()
      set_target_properties(${SUBPROJECT_TARGET} PROPERTIES
        MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL"
      )
    endif()

    target_compile_options(${SUBPROJECT_TARGET} PRIVATE
      $<$<CONFIG:Debug>:
      ${DEFAULT_CXX_OPTIMIZATION_DEBUG};
      ${DEFAULT_CXX_DEBUG_SANITIZER};

      # ${DEFAULT_CXX_DEBUG_RUNTIME_LIBRARY};
      ${DEFAULT_CXX_DEBUG_RUNTIME_CHECKS};
      >
      $<$<CONFIG:Release,RelWithDebInfo>:
      ${DEFAULT_CXX_COMPILER_OPTIMIZATION_RELEASE};
      >
      ${DEFAULT_CXX_CALLING_CONVENTION};
      ${DEFAULT_CXX_COMPILER_OPTIONS};
      ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
      ${DEFAULT_CXX_EXCEPTION_HANDLING};
      ${DEFAULT_CXX_FLOATINGPOINT};
      ${DEFAULT_CXX_SECURITY_CHECKS};
      ${DEFAULT_CXX_STANDARDS_CONFORMANCE};
      ${DEFAULT_CXX_WARNING_LEVEL};
    )

    target_link_options(${SUBPROJECT_TARGET} PRIVATE
      $<$<CONFIG:Release,RelWithDebInfo>:
      ${DEFAULT_CXX_LINKER_OPTIMIZATION_RELEASE};
      >
      /SUBSYSTEM:CONSOLE;
    )

    if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x64")
      target_link_options(${SUBPROJECT_TARGET} PRIVATE
        /MACHINE:X64;
      )
    elseif("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "win32" OR "${CMAKE_VS_PLATFORM_NAME}" STREQUAL "Win32")
      target_link_options(${SUBPROJECT_TARGET} PRIVATE
        /MACHINE:X86;
      )
    endif()
  endif()
endfunction()

create_project(nanoid)

if(MSVC)
  create_project(nanoid-static static)
endif()

# ############################################
# Unit tests
enable_testing()
add_executable(nanoid_tests EXCLUDE_FROM_ALL tests/unit_tests.cpp)

if(MSVC)
      set_target_properties(nanoid_tests PROPERTIES
        MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL"
      )


    target_compile_options(nanoid_tests PRIVATE
      $<$<CONFIG:Debug>:
      ${DEFAULT_CXX_OPTIMIZATION_DEBUG};
      ${DEFAULT_CXX_DEBUG_SANITIZER};

      # ${DEFAULT_CXX_DEBUG_RUNTIME_LIBRARY};
      ${DEFAULT_CXX_DEBUG_RUNTIME_CHECKS};
      >
      $<$<CONFIG:Release,RelWithDebInfo>:
      ${DEFAULT_CXX_COMPILER_OPTIMIZATION_RELEASE};
      >
      ${DEFAULT_CXX_CALLING_CONVENTION};
      ${DEFAULT_CXX_COMPILER_OPTIONS};
      ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
      ${DEFAULT_CXX_EXCEPTION_HANDLING};
      ${DEFAULT_CXX_FLOATINGPOINT};
      ${DEFAULT_CXX_SECURITY_CHECKS};
      ${DEFAULT_CXX_STANDARDS_CONFORMANCE};
      # ${DEFAULT_CXX_WARNING_LEVEL};
    )

    target_link_options(nanoid_tests PRIVATE
      $<$<CONFIG:Release,RelWithDebInfo>:
      ${DEFAULT_CXX_LINKER_OPTIMIZATION_RELEASE};
      >
      /SUBSYSTEM:CONSOLE;
    )

    if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x64")
      target_link_options(nanoid_tests PRIVATE
        /MACHINE:X64;
      )
    elseif("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "win32" OR "${CMAKE_VS_PLATFORM_NAME}" STREQUAL "Win32")
      target_link_options(nanoid_tests PRIVATE
        /MACHINE:X86;
      )
    endif()
  endif()

target_link_libraries(nanoid_tests
  nanoid
  Catch2::Test
)
add_test(test_all nanoid_tests)
set_target_properties(nanoid_tests PROPERTIES FOLDER "third-party")
