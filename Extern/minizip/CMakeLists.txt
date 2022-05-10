add_library(minizip STATIC)

target_sources(minizip
  PRIVATE
    unzip.c
    ioapi.c)

target_include_directories(minizip
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR})

if (SIMCOUPE_PORTABLE)
  target_link_libraries(minizip PUBLIC libz.a)
else()
  target_link_libraries(minizip PUBLIC ZLIB::ZLIB)
endif()
