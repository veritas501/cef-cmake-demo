# Copyright (c) 2016 The Chromium Embedded Framework Authors. All rights
# reserved. Use of this source code is governed by a BSD-style license that
# can be found in the LICENSE file.

cmake_minimum_required(VERSION 3.16)

# Only generate Debug and Release configuration types.
set(CMAKE_CONFIGURATION_TYPES Debug Release)

# Project name.
project(cef)

# Use folders in the resulting project files.
set_property(GLOBAL PROPERTY OS_FOLDERS ON)

#
# CEF configuration.
#

# Specify the CEF distribution version.
set(CEF_VERSION "95.7.17+g4208276+chromium-95.0.4638.69")

# Determine the platform.
if ("${CMAKE_SYSTEM_NAME}" STREQUAL "Darwin")
  if ("${PROJECT_ARCH}" STREQUAL "arm64")
    set(CEF_PLATFORM "macosarm64")
  else ()
    set(CEF_PLATFORM "macosx64")
  endif ()
elseif ("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
  if (CMAKE_SIZEOF_VOID_P MATCHES 8)
    set(CEF_PLATFORM "linux64")
  else ()
    set(CEF_PLATFORM "linux32")
  endif ()
elseif ("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  if (CMAKE_SIZEOF_VOID_P MATCHES 8)
    set(CEF_PLATFORM "windows64")
  else ()
    set(CEF_PLATFORM "windows32")
  endif ()
endif ()

message(STATUS "CEF_PLATFORM: ${CEF_PLATFORM}")

# Add this project's cmake/ directory to the module path.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# Download and extract the CEF binary distribution (executes DownloadCEF.cmake).
include(DownloadCEF)
DownloadCEF("${CEF_PLATFORM}" "${CEF_VERSION}" "${CMAKE_CURRENT_SOURCE_DIR}")

# Add the CEF binary distribution's cmake/ directory to the module path.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CEF_ROOT}/cmake")

# Load the CEF configuration (executes FindCEF.cmake).
find_package(CEF REQUIRED)

#
# Target configuration.
#

# Include the libcef_dll_wrapper target (executes libcef_dll/CMakeLists.txt).
add_subdirectory(${CEF_LIBCEF_DLL_WRAPPER_PATH} libcef_dll_wrapper)

# Display configuration settings.
PRINT_CEF_CONFIG()
