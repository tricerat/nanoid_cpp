cmake_minimum_required(VERSION 3.5)
project (nanoid_cpp
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
  #target_compile_options(nanoid PRIVATE
  #  $<$<CXX_COMPILER_ID:MSVC>:/W4>
  #  $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-Wall -Wextra -pedantic>
  #)

  if (ARGV1 STREQUAL "static")
    set_target_properties(${SUBPROJECT_TARGET} PROPERTIES
      MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
    )
  else()
    set_target_properties(${SUBPROJECT_TARGET} PROPERTIES
      MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL"
    )
  endif()

  set_target_properties(${SUBPROJECT_TARGET} PROPERTIES FOLDER "third-party")
  set_property(TARGET ${SUBPROJECT_TARGET} PROPERTY POSITION_INDEPENDENT_CODE ON)
endfunction()

create_project(nanoid)
if(MSVC)
create_project(nanoid-static static)
endif()

#############################################
# Unit tests

enable_testing()
add_executable(nanoid_tests EXCLUDE_FROM_ALL tests/unit_tests.cpp)

target_link_libraries(nanoid_tests
    nanoid
    Catch2::Test
)
add_test(test_all nanoid_tests)
set_target_properties(nanoid_tests PROPERTIES FOLDER "third-party")
