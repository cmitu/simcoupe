include(FetchContent)

#set(CMAKE_THREAD_PREFER_PTHREAD pthread)
#find_package(Threads REQUIRED)
#target_link_libraries(${PROJECT_NAME} PRIVATE Threads::Threads)

find_package(ZLIB)
if (ZLIB_FOUND)
  set(HAVE_LIBZ 1 PARENT_SCOPE)
  add_subdirectory(minizip)
  target_link_libraries(${PROJECT_NAME} PRIVATE minizip)
endif()

find_package(BZip2)
if (BZIP2_FOUND)
  if (SIMCOUPE_PORTABLE)
    target_link_libraries(${PROJECT_NAME} PRIVATE libbz2.a)
  else()
    target_link_libraries(${PROJECT_NAME} PRIVATE BZip2::BZip2)
  endif()
  set(HAVE_LIBBZ2 1 PARENT_SCOPE)
endif()

if (NOT APPLE)
  check_include_file_cxx("filesystem" HAVE_STD_FILESYSTEM)
endif()
if (NOT HAVE_STD_FILESYSTEM)
  add_subdirectory(ghc)
  target_link_libraries(${PROJECT_NAME} PRIVATE ghc_filesystem)
endif()

if (WIN32)
  add_subdirectory(fdrawcmd)
  target_link_libraries(${PROJECT_NAME} PRIVATE fdrawcmd)
endif()

add_subdirectory(blip_buffer)
target_link_libraries(${PROJECT_NAME} PRIVATE blip_buffer)

add_subdirectory(gl3w)
target_link_libraries(${PROJECT_NAME} PRIVATE gl3w)

add_subdirectory(imgui)
target_link_libraries(${PROJECT_NAME} PRIVATE imgui)

add_subdirectory(sp0256)
target_link_libraries(${PROJECT_NAME} PRIVATE sp0256)

message(STATUS "Fetching fmt project")
FetchContent_Declare(
  fmt
  GIT_REPOSITORY https://github.com/fmtlib/fmt.git
  GIT_TAG b6f4ceaed0a0a24ccf575fab6c56dd50ccf6f1a9)
set(FMT_INSTALL OFF)
FetchContent_MakeAvailable(fmt)
target_link_libraries(${PROJECT_NAME} PRIVATE fmt)
target_include_directories(${PROJECT_NAME} PRIVATE ${fmt_SOURCE_DIR}/include)

message(STATUS "Fetching libspectrum project")
FetchContent_Declare(
  spectrum
  GIT_REPOSITORY https://github.com/simonowen/libspectrum.git
  GIT_TAG eb93bd36f6253471bf709bb9745176d1f9c994f0)
FetchContent_MakeAvailable(spectrum)
target_link_libraries(${PROJECT_NAME} PRIVATE spectrum)
target_include_directories(${PROJECT_NAME} PRIVATE ${spectrum_SOURCE_DIR})

message(STATUS "Fetching resid project")
FetchContent_Declare(
  resid
  GIT_REPOSITORY https://github.com/simonowen/resid.git
  GIT_TAG 65248e7eb30e037b51ffb061374b3dbd3811308c)
FetchContent_MakeAvailable(resid)
target_link_libraries(${PROJECT_NAME} PRIVATE resid)

message(STATUS "Fetching SAASound project")
FetchContent_Declare(
  saasound
  GIT_REPOSITORY https://github.com/simonowen/SAASound.git
  GIT_TAG 8a143060f0acbd28022bd959a5235b2ab2b64c01)
FetchContent_MakeAvailable(saasound)
target_link_libraries(${PROJECT_NAME} PRIVATE SAASound)
target_include_directories(${PROJECT_NAME} PRIVATE ${saasound_SOURCE_DIR}/include)

message(STATUS "Fetching whereami project")
FetchContent_Declare(
  whereami
  GIT_REPOSITORY https://github.com/gpakosz/whereami.git
  GIT_TAG ba364cd54fd431c76c045393b6522b4bff547f50)
FetchContent_Populate(whereami)
add_library(whereami STATIC ${whereami_SOURCE_DIR}/src/whereami.c)
target_include_directories(whereami PUBLIC ${whereami_SOURCE_DIR}/src)
target_link_libraries(${PROJECT_NAME} PRIVATE whereami)

message(STATUS "Fetching z80 project")
FetchContent_Declare(
  z80
  GIT_REPOSITORY https://github.com/kosarev/z80.git
  GIT_TAG 1e25746267d18ddc4684a5359effd6bb4bccc042)
FetchContent_Populate(z80)
add_library(z80 INTERFACE)
target_include_directories(z80 INTERFACE ${z80_SOURCE_DIR})
target_link_libraries(${PROJECT_NAME} PRIVATE z80)

if (BUILD_SDL)
  if (SIMCOUPE_PORTABLE)
    message(STATUS "Fetching SDL2 project")
    FetchContent_Declare(
      sdl2
      GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
      GIT_TAG        release-2.0.20
      GIT_PROGRESS   TRUE)
    set(SDL_SHARED OFF)
    FetchContent_MakeAvailable(sdl2)
    target_link_libraries(${PROJECT_NAME} PRIVATE SDL2main SDL2-static)
  else()
    find_package(SDL2 REQUIRED)
    target_include_directories(${PROJECT_NAME} PRIVATE ${SDL2_INCLUDE_DIR})
    target_link_libraries(${PROJECT_NAME} PRIVATE ${SDL2_LIBRARY})
  endif()
  set(HAVE_LIBSDL2 1 PARENT_SCOPE)

  find_package(OpenGL)
  if (OPENGL_FOUND)
    target_link_libraries(${PROJECT_NAME} PRIVATE ${OPENGL_LIBRARIES})
    set(HAVE_OPENGL 1 PARENT_SCOPE)
  endif()
endif()

if (EXISTS "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/include/xaudio2redist/xaudio2.h")
  target_include_directories(
    ${PROJECT_NAME} PRIVATE
    "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/include/xaudio2redist")
  target_link_libraries(
    ${PROJECT_NAME} PRIVATE
    "${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET}/lib/xaudio2_9redist.lib")

  message(STATUS "Found xaudio2redist (vcpkg)")
  set(HAVE_XAUDIO2REDIST 1 PARENT_SCOPE)
elseif (WIN32)
  message(STATUS "xaudio2redist (vcpkg) not found")
endif()

if (NOT MSVC)
  check_cxx_compiler_flag("-stdlib=libstdc++" COMPILER_SUPPORTS_STDLIBCXX)
  if (COMPILER_SUPPORTS_STDLIBCXX AND NOT APPLE)
    target_compile_options(${PROJECT_NAME} PRIVATE $<$<COMPILE_LANGUAGE:CXX>:-stdlib=libstdc++>)
  endif()
endif()
