add_library(blip_buffer STATIC)

target_compile_features(blip_buffer PRIVATE cxx_std_17)

target_sources(blip_buffer
  PRIVATE
    Blip_Buffer.cpp)

target_include_directories(blip_buffer
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR})
