add_library(sp0256 STATIC)

target_compile_features(sp0256 PRIVATE cxx_std_17)

target_sources(sp0256
  PRIVATE
    sp0256.cpp)

target_include_directories(sp0256
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR})
