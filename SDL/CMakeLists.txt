target_sources(${PROJECT_NAME}
  PRIVATE
    Audio.cpp
    IDEDisk.cpp
    Input.cpp
    MIDI.cpp
    OSD.cpp
    SDL20.cpp
    SDL20_GL3.cpp
    UI.cpp)

target_include_directories(${PROJECT_NAME}
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR})

if (APPLE)
  add_subdirectory(OSX)
endif()
