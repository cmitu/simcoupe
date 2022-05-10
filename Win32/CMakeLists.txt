add_subdirectory(Shaders)

configure_file(
  version.rc.in
  ${CMAKE_CURRENT_BINARY_DIR}/version.rc)

target_include_directories(${PROJECT_NAME}
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR})

target_sources(${PROJECT_NAME}
  PRIVATE
    Audio.cpp
    D3D11Video.cpp
    Floppy.cpp
    IDEDisk.cpp
    Input.cpp
    MIDI.cpp
    ODMenu.cpp
    OSD.cpp
    UI.cpp
    SimCoupe.rc
    Custom.manifest)

if (CMAKE_SYSTEM_VERSION STREQUAL "6.1")
  target_compile_definitions(${PROJECT_NAME}
    PRIVATE
      _WIN32_WINNT=0x0601)
endif()

target_link_libraries(${PROJECT_NAME}
  PRIVATE
    win32_shaders
    winmm
    comctl32
    shlwapi
    dinput8
    dxguid)
