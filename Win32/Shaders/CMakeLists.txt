add_library(win32_shaders INTERFACE)

target_sources(win32_shaders
  PRIVATE
    D3D11_Aspect_VS.hlsl
    D3D11_Copy_VS.hlsl
    D3D11_Blend_PS.hlsl
    D3D11_Palette_PS.hlsl
    D3D11_Sample_PS.hlsl)

target_include_directories(win32_shaders
  INTERFACE
    ${CMAKE_CURRENT_BINARY_DIR})

set_shader_compiler(D3D11_Aspect_VS.hlsl vs_4_0)
set_shader_compiler(D3D11_Copy_VS.hlsl vs_4_0)
set_shader_compiler(D3D11_Blend_PS.hlsl ps_4_0)
set_shader_compiler(D3D11_Palette_PS.hlsl ps_4_0)
set_shader_compiler(D3D11_Sample_PS.hlsl ps_4_0)
