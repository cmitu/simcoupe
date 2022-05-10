add_library(gl3w STATIC)

target_sources(gl3w PRIVATE src/gl3w.c)

target_link_libraries(gl3w PRIVATE ${CMAKE_DL_LIBS})

target_include_directories(gl3w
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include)
