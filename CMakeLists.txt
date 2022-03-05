cmake_minimum_required(VERSION 3.5)
project(serial
  LANGUAGES CXX
  VERSION 1.2.2
)
set(TARGET_NAME ${PROJECT_NAME}Targets)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif(NOT CMAKE_BUILD_TYPE)
option(SERIAL_BUILD_TESTS "Build tests." OFF)

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
)

if(APPLE)
    # If OSX
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} STATIC ${serial_SRCS})

if(APPLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} PRIVATE rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} PRIVATE setupapi)
endif()

target_include_directories(${PROJECT_NAME}
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
set_target_properties(${PROJECT_NAME}
  PROPERTIES
    VERSION ${PROJECT_VERSION}
)
set(INSTALL_CMAKEDIR share/cmake/${PROJECT_NAME})

install(
  TARGETS
    ${PROJECT_NAME}
  EXPORT ${TARGET_NAME}
  LIBRARY
    COMPONENT lib
  ARCHIVE
    COMPONENT lib
)

install(
  DIRECTORY
    "${CMAKE_SOURCE_DIR}/include/"
  TYPE INCLUDE
  PATTERN "impl" EXCLUDE
)

install(
  EXPORT
    ${TARGET_NAME}
  NAMESPACE
    "${PROJECT_NAME}::"
  DESTINATION
    ${INSTALL_CMAKEDIR}
  COMPONENT
    dev
)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
      COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
  ${CMAKE_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${INSTALL_CMAKEDIR}
)

install(
  FILES
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
  DESTINATION
    ${INSTALL_CMAKEDIR}
)

## Uncomment for example
add_executable(serial_example examples/serial_example.cc)
add_dependencies(serial_example ${PROJECT_NAME})
target_link_libraries(serial_example ${PROJECT_NAME})

## Tests
if(SERIAL_BUILD_TESTS)
if(UNIX)
  include(GoogleTest)
  include(CTest)
  find_program(MEMORYCHECK_COMMAND NAMES valgrind)
  set(MEMORYCHECK_COMMAND_OPTIONS "--trace-children=yes --leak-check=full")

  add_executable(${PROJECT_NAME}-test tests/unix_serial_tests.cc)
  target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME} gtest)
  if(NOT APPLE)
    target_link_libraries(${PROJECT_NAME}-test util)
  endif()
  gtest_discover_tests(${PROJECT_NAME}-test)

  if(NOT APPLE)
    add_executable(${PROJECT_NAME}-test-timer tests/unit/unix_timer_tests.cc)
    target_link_libraries(${PROJECT_NAME}-test-timer ${PROJECT_NAME} gtest)
    gtest_discover_tests(${PROJECT_NAME}-test-timer)
  endif()
endif(UNIX)
endif(SERIAL_BUILD_TESTS)
