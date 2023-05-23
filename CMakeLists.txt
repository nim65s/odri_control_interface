#
# Copyright (c) 2020, New York University and Max Planck Gesellschaft.
#
# License BSD-3 clause
#

cmake_minimum_required(VERSION 3.1)

# ----------------------------------------------------
# --- CXX FLAGS --------------------------------------
# ----------------------------------------------------

set(CXX_DISABLE_WERROR True)
set(CMAKE_VERBOSE_MAKEFILE True)

# These variables have to be defined before running SETUP_PROJECT
set(PROJECT_NAME odri_control_interface)
set(PROJECT_DESCRIPTION
    "Common interface for controlling robots build with the odri master board.")
set(PROJECT_URL
    https://github.com/open-dynamic-robot-initiative/odri_control_interface)
set(PROJECT_USE_CMAKE_EXPORT TRUE)

# --- OPTIONS ----------------------------------------
option(BUILD_PYTHON_INTERFACE "Build the python binding" ON)
option(PYTHON_STANDARD_LAYOUT "Enable standard Python package layout" ON)
option(PYTHON_DEB_LAYOUT "Enable Debian-style Python package layout" OFF)

# Check if the submodule cmake have been initialized
set(JRL_CMAKE_MODULES "${CMAKE_CURRENT_LIST_DIR}/cmake")
if(NOT EXISTS "${CMAKE_SOURCE_DIR}/cmake/base.cmake")
  if(${CMAKE_VERSION} VERSION_LESS "3.14.0")
    message(
      FATAL_ERROR
        "\nPlease run the following command first:\ngit submodule update --init\n"
    )
  else()
    message(STATUS "JRL cmakemodules not found. Let's fetch it.")
    include(FetchContent)
    FetchContent_Declare(
      "jrl-cmakemodules"
      GIT_REPOSITORY "https://github.com/jrl-umi3218/jrl-cmakemodules.git")
    FetchContent_MakeAvailable("jrl-cmakemodules")
    FetchContent_GetProperties("jrl-cmakemodules" SOURCE_DIR JRL_CMAKE_MODULES)
  endif()
endif()

include("${JRL_CMAKE_MODULES}/base.cmake")
include("${JRL_CMAKE_MODULES}/boost.cmake")
include("${JRL_CMAKE_MODULES}/python.cmake")
include("${JRL_CMAKE_MODULES}/ide.cmake")

set_default_cmake_build_type("RelWithDebInfo")

compute_project_args(PROJECT_ARGS LANGUAGES CXX)
project(${PROJECT_NAME} ${PROJECT_ARGS})
check_minimal_cxx_standard(14 ENFORCE)

# ----------------------------------------------------
# --- DEPENDENCIES -----------------------------------
# ----------------------------------------------------

add_project_dependency(yaml-cpp CONFIG REQUIRED)
add_project_dependency(Eigen3 REQUIRED PKG_CONFIG_REQUIRES "eigen3 >= 3.0.5")
add_project_dependency(master_board_sdk REQUIRED)

# Set component to fetch from boost Get the python interface for the bindings
if(BUILD_PYTHON_INTERFACE)
  add_project_dependency(eigenpy 2.7.10 REQUIRED PKG_CONFIG_REQUIRES
                         "eigenpy >= 2.7.10")
endif(BUILD_PYTHON_INTERFACE)

# ----------------------------------------------------
# --- INCLUDE ----------------------------------------
# ----------------------------------------------------

# --- MAIN LIBRARY -------------------------------------------------------------
set(ODRI_CONTROL_INTERFACE_SRC src/joint_modules.cpp src/imu.cpp src/robot.cpp
                               src/calibration.cpp src/utils.cpp)
add_library(${PROJECT_NAME} SHARED ${ODRI_CONTROL_INTERFACE_SRC})
target_link_libraries(${PROJECT_NAME} yaml-cpp)
target_link_libraries(${PROJECT_NAME} master_board_sdk::master_board_sdk)
target_link_libraries(${PROJECT_NAME} Eigen3::Eigen)
target_include_directories(
  ${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
                         $<INSTALL_INTERFACE:include>)
get_relative_rpath(lib ${PROJECT_NAME}_INSTALL_RPATH)
set_target_properties(${PROJECT_NAME} PROPERTIES INSTALL_RPATH "${${PROJECT_NAME}_INSTALL_RPATH}")

install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${TARGETS_EXPORT_NAME}
  DESTINATION lib)
install(DIRECTORY include/ DESTINATION include)

# --- BINDINGS -----------------------------------------------------------------

if(BUILD_PYTHON_INTERFACE)
  # --- Setup the wrapper name and source files --- #
  set(PYWRAP ${PROJECT_NAME}_pywrap)
  set(${PYWRAP}_HEADERS srcpy/bindings.h)
  set(${PYWRAP}_SOURCES srcpy/bindings.cpp)

  # --- Build the wrapper library --- #
  set(${PYWRAP}_INSTALL_DIR ${PYTHON_SITELIB})
  add_library(${PYWRAP} SHARED ${${PYWRAP}_SOURCES} ${${PYWRAP}_HEADERS})
  target_include_directories(${PYWRAP} SYSTEM PRIVATE ${PYTHON_INCLUDE_DIR})
  target_link_libraries(${PYWRAP} ${PROJECT_NAME} eigenpy::eigenpy)
  target_link_boost_python(${PYWRAP})
  set_target_properties(${PYWRAP} PROPERTIES INSTALL_RPATH
                                             "${CMAKE_INSTALL_PREFIX}/lib")
  get_relative_rpath(${${PYWRAP}_INSTALL_DIR} ${PYWRAP}_INSTALL_RPATH)
  set_target_properties(${PYWRAP} PROPERTIES INSTALL_RPATH "${${PYWRAP}_INSTALL_RPATH}")
  install(TARGETS ${PYWRAP} DESTINATION ${${PYWRAP}_INSTALL_DIR})

  # --- Allow to do: make python --- #
  add_custom_target(python)
  set_target_properties(python PROPERTIES EXCLUDE_FROM_DEFAULT_BUILD True)
  add_dependencies(python ${PYWRAP})

endif(BUILD_PYTHON_INTERFACE)

# --- CHECK DEMOS -----------------------------------------------------------
macro(create_demo source)

  set(demo_name ${PROJECT_NAME}_${source})
  add_executable(${demo_name} demos/${source}.cpp)
  target_link_libraries(${demo_name} ${PROJECT_NAME})
  target_compile_definitions(
    ${demo_name}
    PUBLIC CONFIG_SOLO12_YAML="${PROJECT_SOURCE_DIR}/demos/config_solo12.yaml"
    PUBLIC
      CONFIG_TESTBENCH_YAML="${PROJECT_SOURCE_DIR}/demos/config_testbench.yaml")
  get_relative_rpath(bin ${demo_name}_INSTALL_RPATH)
  set_target_properties(${demo_name} PROPERTIES INSTALL_RPATH "${${demo_name}_INSTALL_RPATH}")
  install(TARGETS ${demo_name} DESTINATION bin)

endmacro(create_demo source)

create_demo(demo_solo12)
create_demo(demo_testbench)
