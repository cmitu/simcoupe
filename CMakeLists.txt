cmake_minimum_required(VERSION 3.14)
cmake_policy(SET CMP0072 NEW) # prefer GLVND for OpenGL

project(simcoupe)
set(PROJECT_TITLE SimCoupe)

add_executable(${PROJECT_NAME} WIN32 MACOSX_BUNDLE)
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_17)

set(MAJOR_VERSION 1)
set(MINOR_VERSION 2)
set(PATCH_VERSION 11)

set(BUILD_BACKEND "auto" CACHE STRING "Back-end framework for video/sound/input")
set_property(CACHE BUILD_BACKEND PROPERTY STRINGS auto win32 sdl)

if (BUILD_BACKEND STREQUAL "win32" OR (BUILD_BACKEND STREQUAL "auto" AND WIN32))
  set(BUILD_WIN32 1)
elseif (BUILD_BACKEND STREQUAL "sdl" OR BUILD_BACKEND STREQUAL "auto")
  set(BUILD_SDL 1)
  set(MIN_SDL_VERSION 2.0.17)
else()
  message(FATAL_ERROR "Unknown backend type: ${BUILD_BACKEND}")
endif()
message(STATUS "Build back-end: ${BUILD_BACKEND}")

if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
include(GNUInstallDirs)
include(PreCompiledHeaders)
include(CheckCXXCompilerFlag)
include(CheckIncludeFileCXX)
include(HlslCompile)

if (CMAKE_TOOLCHAIN_FILE)
  message(STATUS "Using toolchain file: ${CMAKE_TOOLCHAIN_FILE}")
  include(${CMAKE_TOOLCHAIN_FILE})
endif()

set(RESOURCE_DIR ${CMAKE_INSTALL_FULL_DATAROOTDIR}/${PROJECT_NAME})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
set(CMAKE_RC_FLAGS "-nologo ${CMAKE_RC_FLAGS}")

set(RESOURCE_FILES
  ReadMe.md
  Manual.md
  ChangeLog.md)

target_sources(${PROJECT_NAME}
  PRIVATE
    ${RESOURCE_FILES})

if (APPLE)
  get_target_property(PREV_RESOURCE_FILES ${PROJECT_NAME} RESOURCE)
  set(RESOURCE_FILES ${PREV_RESOURCE_FILES} ${RESOURCE_FILES})

  set_target_properties(${PROJECT_NAME}
    PROPERTIES
      RESOURCE "${RESOURCE_FILES}")
endif()

add_subdirectory(Base)
add_subdirectory(Resource)
add_subdirectory(Extern EXCLUDE_FROM_ALL)

if (BUILD_WIN32)
  add_subdirectory(Win32)
elseif (BUILD_SDL)
  add_subdirectory(SDL)

  if (WIN32)
    target_sources(${PROJECT_NAME}
      PRIVATE
        Win32/Floppy.cpp
        Win32/IconOnly.rc)
  endif()
endif()

if (WIN32)
  set_target_properties(${PROJECT_NAME}
    PROPERTIES
      OUTPUT_NAME ${PROJECT_TITLE})
endif()

if (HAVE_XAUDIO2REDIST)
  add_custom_command(
    TARGET ${PROJECT_NAME}
    POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
      "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/bin/xaudio2_9redist.dll"
      $<TARGET_FILE_DIR:${PROJECT_NAME}>)
endif()

configure_file(
  config.h.in
  ${CMAKE_CURRENT_BINARY_DIR}/config.h)

target_include_directories(${PROJECT_NAME}
  PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR})

if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  install(
    FILES package/simcoupe.desktop
    DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/applications)
  install(
    FILES package/simcoupe.png
    DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/128x128/apps)
endif()

install(
  FILES ${RESOURCE_FILES}
  DESTINATION ${RESOURCE_DIR})

install(
  TARGETS ${PROJECT_NAME}
  DESTINATION ${CMAKE_INSTALL_BINDIR})

include(cpack_simcoupe)
