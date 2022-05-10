add_library(imgui STATIC)

target_compile_features(imgui PRIVATE cxx_std_17)

target_sources(imgui
  PRIVATE
    imgui.cpp
    imgui_draw.cpp
    imgui_widgets.cpp
    imgui_tables.cpp
    imgui_demo.cpp
  PUBLIC
    imgui.h)

if (BUILD_WIN32)
  target_sources(imgui
    PRIVATE
      backends/ImGui_VS.hlsl
      backends/ImGui_PS.hlsl
      backends/imgui_impl_dx11.cpp
      backends/imgui_impl_win32.cpp)

  target_include_directories(imgui
    PRIVATE
      ${CMAKE_CURRENT_BINARY_DIR})

  set_shader_compiler(backends/ImGui_VS.hlsl vs_4_0)
  set_shader_compiler(backends/ImGui_PS.hlsl ps_4_0)
endif()

if(BUILD_SDL)
  target_sources(imgui
    PRIVATE
      backends/imgui_impl_opengl3.cpp
      backends/imgui_impl_sdl.cpp
      backends/imgui_impl_sdlrenderer.cpp)

  if (SIMCOUPE_PORTABLE)
    target_link_libraries(imgui PRIVATE SDL2-static)
  else()
    find_package(SDL2 REQUIRED)
    target_include_directories(imgui PRIVATE ${SDL2_INCLUDE_DIR})
  endif()
endif()

target_include_directories(imgui
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR})
