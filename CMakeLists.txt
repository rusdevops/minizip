#***************************************************************************
# Copyright:  Matthias Schmieder,
# E-Mail:     schmieder.matthias@gmail.com
# Year:       2016
#***************************************************************************
cmake_minimum_required(VERSION 2.8)


# Set a consistent MACOSX_RPATH default across all CMake versions. 
# When CMake 2.8.12 is required, change this default to 1.
# When CMake 3.0.0 is required, remove this block (see CMP0042).
if(NOT DEFINED CMAKE_MACOSX_RPATH)
  set(CMAKE_MACOSX_RPATH 0)
endif()

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.14.26.tar.gz"
    SHA1 "45ac03944db2bc7fc55dfceca42d89ec7fa1dca2"
)

project("minizip")

hunter_add_package(ZLIB)
find_package(ZLIB CONFIG REQUIRED)

set(MINIZIP_SRC "ioapi.c"
  "ioapi_buf.c"
  "ioapi_mem.c"
  "unzip.c"
  "zip.c")

set(MINIZIP_PUBLIC_HEADERS "crypt.h"
  "ioapi.h"
  "ioapi_buf.h"
  "ioapi_mem.h"
  "unzip.h"
  "zip.h")

if(WIN32)
  list(APPEND MINIZIP_SRC "iowin32.c")
  list(APPEND MINIZIP_PUBLIC_HEADERS "iowin32.h")
endif()

# create minizip library
add_library(minizip ${MINIZIP_SRC} ${MINIZIP_PUBLIC_HEADERS})

target_link_libraries(minizip PUBLIC ZLIB::zlib)

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")

# Configuration
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

option(USE_AES "enables building of aes library" ON)
if(USE_AES)
  set(AES_SRC 
    aes/aescrypt.c
    aes/aeskey.c
    aes/aestab.c
    aes/entropy.c
    aes/fileenc.c
    aes/hmac.c
    aes/prng.c
    aes/pwd2key.c
    aes/sha1.c)

  set(AES_PUBLIC_HEADERS 
    aes/aes.h
    aes/aes_via_ace.h
    aes/aesopt.h
    aes/aestab.h
    aes/brg_endian.h
    aes/brg_types.h
    aes/entropy.h
    aes/fileenc.h
    aes/hmac.h
    aes/prng.h
    aes/pwd2key.h
    aes/sha1.h)

  add_library(aes ${AES_SRC} ${AES_PUBLIC_HEADERS})

  target_compile_definitions(aes PUBLIC "HAVE_AES")

  target_link_libraries(minizip PUBLIC aes)

  install(
    TARGETS aes
    EXPORT "${targets_export_name}"
    INCLUDES DESTINATION "include"
    RUNTIME DESTINATION  "bin"
    LIBRARY DESTINATION  "lib"
    ARCHIVE DESTINATION  "lib"
  )

  install(FILES ${AES_PUBLIC_HEADERS}
    DESTINATION "include/minizip/aes")
endif()

install(
  TARGETS minizip
  EXPORT "${targets_export_name}"
  INCLUDES DESTINATION "include"
  RUNTIME DESTINATION "bin"
  LIBRARY DESTINATION "lib"
  ARCHIVE DESTINATION "lib"
)

install(EXPORT "${targets_export_name}"
        DESTINATION "${config_install_dir}"
        NAMESPACE "${namespace}")

include(CMakePackageConfigHelpers)

configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(FILES "${project_config}" DESTINATION "${config_install_dir}")

install(FILES ${MINIZIP_PUBLIC_HEADERS}
  DESTINATION "include/minizip")

option (BUILD_TEST "enabled building of executables minizip and miniunz. Requires ZLIB!" OFF)
if(BUILD_TEST)
  add_executable(miniunz_exec miniunz.c)
  target_link_libraries(miniunz_exec minizip)

  add_executable(minizip_exec minizip.c)
  target_link_libraries(minizip_exec minizip)

  install(TARGETS miniunz_exec minizip_exec
          RUNTIME DESTINATION "bin")
endif()
