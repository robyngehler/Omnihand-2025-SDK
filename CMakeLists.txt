cmake_minimum_required(VERSION 3.24)
project(omni_hand_2025 CXX)

# Set cmake path
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

string(TOLOWER "${CMAKE_SYSTEM_PROCESSOR}" OMNIHAND_SYSTEM_PROCESSOR)
set(OMNIHAND_AARCH64_ARCHES aarch64 arm64)

set(OMNIHAND_DEFAULT_CAN_DRIVER_VALUE "zlg")
if(OMNIHAND_SYSTEM_PROCESSOR IN_LIST OMNIHAND_AARCH64_ARCHES)
  set(OMNIHAND_DEFAULT_CAN_DRIVER_VALUE "socket")
endif()

# Some option
option(BUILD_PYTHON_BINDING "Build Python binding" ON)
option(BUILD_CPP_EXAMPLES "Build CPP examples" ON)
option(BUILD_ROS_NODE "Build ROS2 node packages" ON)
set(OMNIHAND_CAN_DRIVER "${OMNIHAND_DEFAULT_CAN_DRIVER_VALUE}" CACHE STRING "Default CAN driver backend: zlg or socket")
set_property(CACHE OMNIHAND_CAN_DRIVER PROPERTY STRINGS zlg socket)
set(OMNIHAND_ZLG_SDK_PATH "" CACHE PATH "Path to a ZLG userspace SDK directory containing zcan.h and libusbcanfd.so")

set(OMNIHAND_ZLG_BUNDLED_SDK_PATH "${CMAKE_CURRENT_SOURCE_DIR}/thirdParty/usbcanfd_libusb_x64_1.0.10_250328")
set(OMNIHAND_ZLG_EFFECTIVE_PATH "")

if(OMNIHAND_CAN_DRIVER STREQUAL "zlg")
  if(OMNIHAND_ZLG_SDK_PATH)
    set(OMNIHAND_ZLG_EFFECTIVE_PATH "${OMNIHAND_ZLG_SDK_PATH}")
  elseif(OMNIHAND_SYSTEM_PROCESSOR IN_LIST OMNIHAND_AARCH64_ARCHES)
    message(FATAL_ERROR
      "OMNIHAND_CAN_DRIVER=zlg requires an explicit aarch64-compatible ZLG SDK path on ${CMAKE_SYSTEM_PROCESSOR}. "
      "The bundled thirdParty/usbcanfd_libusb_x64_1.0.10_250328 package is x86_64-only. "
      "Use -DOMNIHAND_CAN_DRIVER=socket for the repo-local Jetson flow, or provide -DOMNIHAND_ZLG_SDK_PATH=/path/to/aarch64/zlg/sdk.")
  else()
    set(OMNIHAND_ZLG_EFFECTIVE_PATH "${OMNIHAND_ZLG_BUNDLED_SDK_PATH}")
  endif()
endif()

# Some necessary settings
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Master project configuration
if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_SOURCE_DIR)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

  set(FETCHCONTENT_BASE_DIR ${CMAKE_SOURCE_DIR}/_deps)

  set(BUILD_SHARED_LIBS OFF)
  set(CMAKE_C_VISIBILITY_PRESET hidden)
  set(CMAKE_C_VISIBILITY_INLINES_HIDDEN ON)
  set(CMAKE_CXX_VISIBILITY_PRESET hidden)
  set(CMAKE_CXX_VISIBILITY_INLINES_HIDDEN ON)

  if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
    if(AIMRT_MUJOCO_SIM_BUILD_TESTS)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --coverage")
      set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} --coverage")
    endif()
  endif()

  set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
endif()

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include(GetYamlCpp)
include(GetSerial)

# find python dependencies
if(BUILD_PYTHON_BINDING)
  find_package(
    Python3
    COMPONENTS Interpreter Development
    REQUIRED)
  find_package(pybind11 QUIET)
  if(NOT pybind11_FOUND)
    message(STATUS "System pybind11 not found, will download from git...")
    include(GetPybind11)
  endif()

  include(CheckPythonPackage)
  check_python_package(build BUILD_FOUND)
  check_python_package(setuptools SETUPTOOLS_FOUND)
  check_python_package(wheel WHEEL_FOUND)

  if(NOT BUILD_FOUND
     OR NOT SETUPTOOLS_FOUND
     OR NOT WHEEL_FOUND)
    message(WARNING "Can not find build, setuptools or wheel in your python environment, will not generate aimrt_py package! ")
    message(WARNING "Try to install build, setuptools and wheel by `pip3 install build setuptools wheel --upgrade`")
  endif()
endif()

add_subdirectory(src)

if(BUILD_ROS_NODE)
  add_subdirectory(node)
endif()

if(BUILD_PYTHON_BINDING)
  add_subdirectory(python)
endif()

if(BUILD_CPP_EXAMPLES)
  add_subdirectory(examples)
endif()
