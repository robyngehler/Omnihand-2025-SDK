# Copyright (c) 2025, Agibot Co., Ltd.
# OmniHand 2025 SDK is licensed under Mulan PSL v2


set(CUR_TARGET_NAME "omnihand_2025_core")

set(PYTHON_PKG_DIR ${CMAKE_BINARY_DIR}/omnihand_2025_pkg)

pybind11_add_module(${CUR_TARGET_NAME} SHARED ${CMAKE_CURRENT_SOURCE_DIR}/binding.cc)

target_include_directories(
  ${CUR_TARGET_NAME}
  PRIVATE ${CMAKE_SOURCE_DIR}/src
          ${CMAKE_SOURCE_DIR}/src/can_bus_device
          ${USER_PATH})

target_link_libraries(
  ${CUR_TARGET_NAME}
  PRIVATE omnihand2025)

set_target_properties(${CUR_TARGET_NAME} PROPERTIES
    INSTALL_RPATH "$ORIGIN"
    BUILD_WITH_INSTALL_RPATH TRUE
    LIBRARY_OUTPUT_DIRECTORY ${PYTHON_PKG_DIR}/omnihand_2025
)

add_custom_command(
  TARGET ${CUR_TARGET_NAME}
  PRE_BUILD
  COMMAND ${CMAKE_COMMAND} -E make_directory ${PYTHON_PKG_DIR}
  COMMAND ${CMAKE_COMMAND} -E make_directory ${PYTHON_PKG_DIR}/omnihand_2025)


add_custom_command(
  TARGET ${CUR_TARGET_NAME}
  POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:omnihand2025> ${PYTHON_PKG_DIR}/omnihand_2025/
)

if(OMNIHAND_CAN_DRIVER STREQUAL "zlg")
  add_custom_command(
    TARGET ${CUR_TARGET_NAME}
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy ${OMNIHAND_ZLG_EFFECTIVE_PATH}/libusbcanfd.so.1.0.10 ${PYTHON_PKG_DIR}/omnihand_2025/
    COMMAND ${CMAKE_COMMAND} -E create_symlink libusbcanfd.so.1.0.10 ${PYTHON_PKG_DIR}/omnihand_2025/libusbcanfd.so.1
    COMMAND ${CMAKE_COMMAND} -E create_symlink libusbcanfd.so.1.0.10 ${PYTHON_PKG_DIR}/omnihand_2025/libusbcanfd.so
  )
endif()

if(BUILD_FOUND AND SETUPTOOLS_FOUND AND WHEEL_FOUND)
  add_custom_target(
    create_python_pkg ALL
    COMMAND ${CMAKE_COMMAND} -E make_directory ${PYTHON_PKG_DIR}/omnihand_2025
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/omnihand_2025 ${PYTHON_PKG_DIR}/omnihand_2025
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/setup.py ${CMAKE_CURRENT_SOURCE_DIR}/pyproject.toml ${CMAKE_CURRENT_SOURCE_DIR}/MANIFEST.in ${PYTHON_PKG_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_SOURCE_DIR}/VERSION ${PYTHON_PKG_DIR}/VERSION
    COMMAND ${Python3_EXECUTABLE} -m build --wheel
    WORKING_DIRECTORY ${PYTHON_PKG_DIR}
    DEPENDS ${CUR_TARGET_NAME})
else()
  add_custom_target(
    create_python_pkg ALL
    COMMAND ${CMAKE_COMMAND} -E make_directory ${PYTHON_PKG_DIR}/omnihand_2025
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/omnihand_2025 ${PYTHON_PKG_DIR}/omnihand_2025
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/setup.py ${CMAKE_CURRENT_SOURCE_DIR}/pyproject.toml ${CMAKE_CURRENT_SOURCE_DIR}/MANIFEST.in ${PYTHON_PKG_DIR}
    COMMAND ${CMAKE_COMMAND} -E copy ${PROJECT_SOURCE_DIR}/VERSION ${PYTHON_PKG_DIR}/VERSION
    WORKING_DIRECTORY ${PYTHON_PKG_DIR}
    DEPENDS ${CUR_TARGET_NAME})
endif()
