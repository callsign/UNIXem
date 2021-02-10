cmake_minimum_required(VERSION 3.17)

project(UNIXem)

#set(CMAKE_VERBOSE_MAKEFILE ON)
#set(CMAKE_BUILD_TYPE Debug)
#set(CMAKE_FIND_DEBUG_MODE TRUE)

if (MINGW)
  set(CMAKE_SHARED_LIBRARY_PREFIX "")
  set(CMAKE_IMPORT_LIBRARY_PREFIX "")
endif(MINGW)

# Specify standards
# Use C11 standard
if (NOT DEFINED CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 11)
endif()

# Add c++11 flags
if (NOT DEFINED CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 11)
else()
  if (CMAKE_CXX_STANDARD LESS 11)
    message(FATAL_ERROR "CMAKE_CXX_STANDARD is less than 11, please specify at least SET(CMAKE_CXX_STANDARD 11)")
  endif()
endif()
if (NOT DEFINED CMAKE_CXX_STANDARD_REQUIRED)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()
if (NOT DEFINED CMAKE_CXX_EXTENSIONS)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

if (NOT DEFINED CMAKE_POSITION_INDEPENDENT_CODE)
  set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
endif()

# Base of the CCS source tree
set(UNIXEM_SRC_DIR ${PROJECT_SOURCE_DIR}/src)

include_directories(${PROJECT_SOURCE_DIR}/include)

message(STATUS "C Compiler ....... : ${CMAKE_C_COMPILER}")
message(STATUS "C Standard ....... : ${CMAKE_C_STANDARD}")
message(STATUS "C Flags .......... : ${CMAKE_C_FLAGS}")
message(STATUS "C++ Standard ..... : ${CMAKE_CXX_STANDARD}")
message(STATUS "PIC .............. : ${CMAKE_POSITION_INDEPENDENT_CODE}")

# # We need pthreads for Windows
# if (MINGW)
set(CMAKE_THREAD_LIBS_INIT "-lpthread")
set(CMAKE_HAVE_THREADS_LIBRARY 1)
set(CMAKE_USE_WIN32_THREADS_INIT 0)
set(CMAKE_USE_PTHREADS_INIT 1)
set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)
# endif()

include_directories(${CMAKE_CURRENT_BINARY_DIR})

# BEGIN: Unixem build

list(APPEND unixem_sources
    atomic.c
    dirent.c
    dlfcn.c
    glob.c
    hostname.c
    internal/util.c
    mktemp.c
    mmap.c
    resource.c
    setenv.c
    time.c
    uio.c
    unistd.c
)
list(TRANSFORM unixem_sources PREPEND ${UNIXEM_SRC_DIR}/)

SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

add_library(unixemlib OBJECT ${unixem_sources})
add_library(unixem STATIC $<TARGET_OBJECTS:unixemlib>)

get_cmake_property(_variableNames VARIABLES)
foreach (_variableName ${_variableNames})
    message(STATUS "${_variableName}=${${_variableName}}")
endforeach()

# END: Unixem build

set(CMAKE_INSTALL_DO_STRIP 1)
install(TARGETS unixem DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include DESTINATION ${CMAKE_INSTALL_PREFIX})
