cmake_minimum_required(VERSION 2.8)

project(BigARTM)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

if (WIN32)
  set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
  set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
  set (CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
endif()

set(3RD_PARTY_DIR ${CMAKE_SOURCE_DIR}/3rdparty)

if (WIN32)
  add_definitions("-D_WIN32")
  add_definitions("-DWIN32")
endif()

if (MSVC)
  add_definitions("-D_VARIADIC_MAX=10")
  add_definitions("/wd4251")
  add_definitions("/MP")
  add_definitions("-D_CRT_SECURE_NO_WARNINGS")
  add_definitions("-D_SCL_SECURE_NO_WARNINGS")
else()
  if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release")
  endif()
  message("-- Build type: ${CMAKE_BUILD_TYPE}")
endif()

set(BUILD_STATIC_LIBS ON)
set(BUILD_SHARED_LIBS OFF)
set(BUILD_gflags_LIB ON)
set(BUILD_gflags_nothreads_LIB OFF)
set(rpcz_build_static ON)
set(rpcz_build_shared ON)

# set compiler flags
if(("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang") OR
   ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU"))
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -pthread -fPIC")
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS} -g -Werror")
  set(GFLAGS_INTTYPES_FORMAT "C99")
  set(LINK_DL "dl")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  set(Boost_USE_MULTITHREADED ON)
  set(Boost_USE_STATIC_LIBS ON)
  set(GFLAGS_INTTYPES_FORMAT "VC7")
  include(InstallRequiredSystemLibraries)
  set(LINK_DL "")
else()
  message("-- Warning: BigARTM has not been tested with '${CMAKE_CXX_COMPILER_ID}' compiler.")
endif()

set(Boost_ADDITIONAL_VERSIONS "1.57" "1.57.0" "1.56" "1.56.0" "1.55" "1.55.0" "1.54" "1.54.0" "1.53" "1.53.0" "1.52" "1.52.0" "1.51" "1.51.0" "1.50" "1.50.0" "1.49" "1.49.0" "1.48" "1.48.0" "1.47" "1.47.0" "1.46" "1.46.0" "1.45" "1.45.0" "1.44" "1.44.0" "1.42" "1.42.0" "1.41.0" "1.41" "1.40.0" "1.40")

# find boost
find_package(Boost REQUIRED)
if (NOT Boost_FOUND)
  message(SEND_ERROR "Failed to find boost libraries.")
endif()

set(BIGARTM_BOOST_COMPONENTS thread program_options date_time filesystem iostreams system)
if(Boost_VERSION GREATER 104700)
  set(BIGARTM_BOOST_COMPONENTS ${BIGARTM_BOOST_COMPONENTS} chrono timer)
endif()

find_package(Boost COMPONENTS REQUIRED ${BIGARTM_BOOST_COMPONENTS})
if(NOT Boost_FOUND)
  message(SEND_ERROR "Failed to find required boost libraries.")
  return()
else()
  include_directories(${Boost_INCLUDE_DIRS})
endif()

add_subdirectory(3rdparty)  # gflags must be compiled without -std=c++11

if(("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang") OR
   ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU"))
  include(CheckCXXCompilerFlag)
  CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
  CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
  if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
  else()
    message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
  endif()
endif()

add_subdirectory(src)

install(FILES LICENSE DESTINATION .)
