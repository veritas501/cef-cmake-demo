cmake_minimum_required(VERSION 3.16)
project(cef_cmake_demo)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/release_bin)
set_property(GLOBAL PROPERTY OS_FOLDERS ON)

#
# add 3rd-party
#
add_subdirectory(deps)

#
# configure CEF at top-level CMake
#
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CEF_ROOT}/cmake")
find_package(CEF REQUIRED)
set(CEF_TARGET "${PROJECT_NAME}")
set(CEF_TARGET_OUT_DIR "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")

#
# source code
#
file(
  GLOB_RECURSE SOURCE_FILES
  ${PROJECT_SOURCE_DIR}/src/*
)

#
# target executable
#
add_executable(
  ${PROJECT_NAME}
  ${SOURCE_FILES}
  ${CEF_EXECUTABLE_RESOURCES}
)

SET_EXECUTABLE_TARGET_PROPERTIES(${CEF_TARGET})

add_dependencies(
  ${PROJECT_NAME}
  libcef_dll_wrapper
)

ADD_LOGICAL_TARGET("libcef_lib" "${CEF_LIB_DEBUG}" "${CEF_LIB_RELEASE}")

target_link_libraries(
  ${PROJECT_NAME}
  libcef_lib
  libcef_dll_wrapper
  ${CEF_STANDARD_LIBS}
)

if(USE_SANDBOX)
  # Logical target used to link the cef_sandbox library.
  ADD_LOGICAL_TARGET("cef_sandbox_lib" "${CEF_SANDBOX_LIB_DEBUG}" "${CEF_SANDBOX_LIB_RELEASE}")
  target_link_libraries(${CEF_TARGET} cef_sandbox_lib ${CEF_SANDBOX_STANDARD_LIBS})
endif()

if ("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  # Add the custom manifest files to the executable.
  ADD_WINDOWS_MANIFEST("${CMAKE_CURRENT_SOURCE_DIR}/resources/win" "${CEF_TARGET}" "exe")
endif()

#
# include files
#

target_include_directories(
  ${PROJECT_NAME}
  PRIVATE ${PROJECT_SOURCE_DIR}/include
  PRIVATE ${CEF_ROOT}
)

# Copy binary and resource files to the target output directory.
COPY_FILES("${PROJECT_NAME}" "${CEF_BINARY_FILES}" "${CEF_BINARY_DIR}" "${CEF_TARGET_OUT_DIR}")
COPY_FILES("${PROJECT_NAME}" "${CEF_RESOURCE_FILES}" "${CEF_RESOURCE_DIR}" "${CEF_TARGET_OUT_DIR}")
