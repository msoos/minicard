cmake_minimum_required(VERSION 2.6 FATAL_ERROR)

project(minicard)

#--------------------------------------------------------------------------------------------------
# Configurable options:

option(STATIC_BINARIES "Link binaries statically." ON)
option(USE_SORELEASE   "Use SORELEASE in shared library filename." ON)

#--------------------------------------------------------------------------------------------------
# Library version:

set(minicard_SOMAJOR   2)
set(minicard_SOMINOR   1)
set(minicard_SORELEASE 0)

# Compute VERSION and SOVERSION:
if (USE_SORELEASE)
  set(minicard_VERSION ${minicard_SOMAJOR}.${minicard_SOMINOR}.${minicard_SORELEASE})
else()
  set(minicard_VERSION ${minicard_SOMAJOR}.${minicard_SOMINOR})
endif()
set(minicard_SOVERSION ${minicard_SOMAJOR})

#--------------------------------------------------------------------------------------------------
# Dependencies:

find_package(ZLIB)
include_directories(${ZLIB_INCLUDE_DIR})
include_directories(${minicard_SOURCE_DIR})

#--------------------------------------------------------------------------------------------------
# Compile flags:

add_definitions(-D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS)

#--------------------------------------------------------------------------------------------------
# Build Targets:

set(minicard_LIB_SOURCES
    utils/Options.cc
    utils/System.cc
    minicard/Solver.cc)

add_library(minicard-lib-static STATIC ${minicard_LIB_SOURCES})
add_library(minicard-lib-shared SHARED ${minicard_LIB_SOURCES})

target_link_libraries(minicard-lib-shared ${ZLIB_LIBRARY})
target_link_libraries(minicard-lib-static ${ZLIB_LIBRARY})

add_executable(minicard minicard/Main.cc)

if(STATIC_BINARIES)
  target_link_libraries(minicard minicard-lib-static)
else()
  target_link_libraries(minicard minicard-lib-shared)
endif()

set_target_properties(minicard-lib-static PROPERTIES OUTPUT_NAME "minicard")
set_target_properties(minicard-lib-shared
  PROPERTIES
    OUTPUT_NAME "minicard"
    VERSION ${minicard_VERSION}
    SOVERSION ${minicard_SOVERSION})

set_target_properties(minicard       PROPERTIES OUTPUT_NAME "minicard")

#--------------------------------------------------------------------------------------------------
# Installation targets:

install(TARGETS minicard-lib-static minicard-lib-shared minicard
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)

install(DIRECTORY mtl utils minicard
        DESTINATION include/minicard
        FILES_MATCHING PATTERN "*.h")
