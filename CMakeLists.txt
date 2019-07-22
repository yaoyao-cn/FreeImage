cmake_minimum_required(VERSION 3.0.0)

project(FreeImage)

#set(FI_SRCS_File fipMakefile.srcs)
set(FI_SRCS_File Makefile.srcs CACHE FILEPATH ".srcs file from freeimage source code")
message(STATUS "Generate Project From ${FI_SRCS_File}")

# read file content
file(STRINGS ${FI_SRCS_File} SRCS_File)

# find VER_MAJOR
string(REGEX MATCH "VER_MAJOR = ([0-9.]*);" VER_MAJOR_STR "${SRCS_File}")
set(VER_MAJOR ${CMAKE_MATCH_1})

# find VER_MINOR
string(REGEX MATCH "VER_MINOR = ([0-9.]*);" VER_MINOR_STR "${SRCS_File}")
set(VER_MINOR ${CMAKE_MATCH_1})

message(STATUS "FreeImage Version ${VER_MAJOR}.${VER_MINOR}")

# find SRCS
string(REGEX MATCH "SRCS = ([^;]*);" SRCS_STR "${SRCS_File}")
set(SRCS ${CMAKE_MATCH_1})

# FreeImagePlus.DllMain confilct with FreeImage.DllMain
string(REPLACE "FreeImagePlus.cpp" "" SRCS ${SRCS})

string(REPLACE " " ";" SRCS ${SRCS})


# find INCLS
string(REGEX MATCH "INCLS = ([^;]*);" INCLS_STR "${SRCS_File}")
set(INCLS ${CMAKE_MATCH_1})
if(INCLS)
    # remove missing FreeImage.h file
    string(REPLACE "./Dist/FreeImage.h" "" INCLS ${INCLS})
    string(REPLACE " " ";" INCLS ${INCLS})
endif()

# find INCLUDE
string(REGEX MATCH "INCLUDE = ([^;]*)" INCLUDE_STR "${SRCS_File}")
set(INCLUDE ${CMAKE_MATCH_1})
string(REGEX REPLACE " ?-I" ";" INCLUDE ${INCLUDE})

# source_group
foreach(FILE ${SRCS} ${INCLS})
  get_filename_component(PARENT_DIR "${FILE}" PATH)

  # ignore .
  string(REGEX REPLACE "\\.[/\\\\]" "" GROUP "${PARENT_DIR}")
  string(REPLACE "/" "\\" GROUP "${GROUP}")

  # group into "Source Files" and "Header Files"
  if ("${FILE}" MATCHES ".*\\.cp{0,2}")
    set(GROUP "Source Files\\${GROUP}")
  elseif("${FILE}" MATCHES ".*\\.h")
    set(GROUP "Header Files\\${GROUP}")
  endif()

  source_group("${GROUP}" FILES "${FILE}")
endforeach()

# add library
add_library(FreeImage ${SRCS} ${INCLS})

# config library
target_compile_definitions(FreeImage PRIVATE LIBRAW_NODLL OPJ_STATIC DISABLE_PERF_MEASUREMENT)

if(BUILD_SHARED_LIBS)
    target_compile_definitions(FreeImage PRIVATE FIP_EXPORTS FREEIMAGE_EXPORTS)
else()
    target_compile_definitions(FreeImage PUBLIC FREEIMAGE_LIB)
endif()

target_include_directories(FreeImage PRIVATE ${INCLUDE})
