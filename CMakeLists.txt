# This file is part of the Splinter library.
# Copyright (C) 2012 Bjarne Grimstad (bjarne.grimstad@gmail.com)
#
# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.

project(splinter)
cmake_minimum_required(VERSION 2.8)
set(CMAKE_BUILD_TYPE Release)

if(WIN32)
    set(CMAKE_CXX_FLAGS_RELEASE "-O2")
endif(WIN32)

set(SPLINTER_VERSION_MAJOR 1)
set(SPLINTER_VERSION_MINOR 2)
set(SPLINTER_VERSION ${SPLINTER_VERSION_MAJOR}-${SPLINTER_VERSION_MINOR})

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

include("${CMAKE_SOURCE_DIR}/cmake/c11Support.cmake")

if(UNIX)
    set(DEFAULT_EIGEN_DIRECTORY "/usr/local/include/eigen3")
elseif(WIN32)
    # TODO: This surely can't be the most logical place to look for Eigen
    set(DEFAULT_EIGEN_DIRECTORY "C:/Program Files/Eigen")
endif(UNIX)

set(DEFAULT_HEADER_DIRECTORY "include")
set(DEFAULT_LIBRARY_DIRECTORY "lib")

if(NOT EIGEN_DIRECTORY)
    find_package(Eigen3 REQUIRED) # Attempt to automatically locate Eigen
    if(EIGEN3_FOUND)
        set(EIGEN_DIRECTORY ${EIGEN3_INCLUDE_DIR})
    else(EIGEN3_FOUND)
        set(EIGEN_DIRECTORY ${DEFAULT_EIGEN_DIRECTORY})
    endif(EIGEN3_FOUND)
endif(NOT EIGEN_DIRECTORY)
# CACHE STRING = Display the option with help text in CMakeCache.txt
set(EIGEN_DIRECTORY ${EIGEN_DIRECTORY} CACHE STRING "Directory where the Eigen library is located.")

if(NOT HEADER_DIRECTORY)
    set(HEADER_DIRECTORY ${DEFAULT_HEADER_DIRECTORY})
endif(NOT HEADER_DIRECTORY)
# CACHE STRING = Display the option with help text in CMakeCache.txt
set(HEADER_DIRECTORY ${HEADER_DIRECTORY} CACHE STRING "Absolute path, or, if relative, relative to CMAKE_INSTALL_PREFIX to install the header files.")

if(NOT LIBRARY_DIRECTORY)
    set(LIBRARY_DIRECTORY ${DEFAULT_LIBRARY_DIRECTORY})
endif(NOT LIBRARY_DIRECTORY)
# CACHE STRING = Display the option with help text in CMakeCache.txt
set(LIBRARY_DIRECTORY ${LIBRARY_DIRECTORY} CACHE STRING "Absolute path, or, if relative, relative to CMAKE_INSTALL_PREFIX to install the library file.")

# These are the headers we need for compilation
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${EIGEN_DIRECTORY})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/test)

# These are the sources we need for compilation of the library (not the testing executables)
#aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src SRC_LIST)
set(SRC_LIST
    include/spline.h
    include/bspline.h
    include/bsplinebasis.h
    include/bsplinebasis1d.h
    include/pspline.h
    include/rbfspline.h
    include/datasample.h
    include/datatable.h
    include/generaldefinitions.h
    include/linearsolvers.h
    include/serialize.h
    include/mykroneckerproduct.h
    src/bspline.cpp
    src/bsplinebasis.cpp
    src/bsplinebasis1d.cpp
    src/pspline.cpp
    src/rbfspline.cpp
    src/datasample.cpp
    src/datatable.cpp
    src/mykroneckerproduct.cpp
)

set(SHARED_LIBRARY splinter-${SPLINTER_VERSION})
set(STATIC_LIBRARY splinter-static-${SPLINTER_VERSION})

set(SHARED_LIBRARY_TEST "splinter-shared-test")
set(STATIC_LIBRARY_TEST "splinter-static-test")
set(SERIALIZE_TEST "splinter-serialize-test")
set(TESTING_UTILITIES ${CMAKE_CURRENT_SOURCE_DIR}/test/testingutilities.cpp)

# Add output library: add_library(libname [SHARED | STATIC] sourcelist)
add_library(${SHARED_LIBRARY} SHARED ${SRC_LIST})
add_library(${STATIC_LIBRARY} STATIC ${SRC_LIST})

# Testing executables
add_executable(${SHARED_LIBRARY_TEST} ${CMAKE_CURRENT_SOURCE_DIR}/test/main.cpp ${TESTING_UTILITIES})
target_link_libraries(${SHARED_LIBRARY_TEST} ${SHARED_LIBRARY})
add_executable(${STATIC_LIBRARY_TEST} ${CMAKE_CURRENT_SOURCE_DIR}/test/main.cpp ${TESTING_UTILITIES})
target_link_libraries(${STATIC_LIBRARY_TEST} ${STATIC_LIBRARY})

# Serialize testing executable
add_executable(${SERIALIZE_TEST} ${CMAKE_CURRENT_SOURCE_DIR}/test/serialize.cpp ${TESTING_UTILITIES})
target_link_libraries(${SERIALIZE_TEST} ${STATIC_LIBRARY})

# Install the header files
install(
    DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
    DESTINATION ${HEADER_DIRECTORY}
)

# Install the shared library file
install(
    TARGETS ${SHARED_LIBRARY}
    DESTINATION ${LIBRARY_DIRECTORY}
)
# Install the static library file
install(
    TARGETS ${STATIC_LIBRARY}
    DESTINATION ${LIBRARY_DIRECTORY}
)

# Used to debug warnings
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -pedantic")
# Need to specify c++ std on OSX Yosemite
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -std=c++11")

# Doxygen documentation
find_package(Doxygen)

if(DOXYGEN_FOUND)

  configure_file(
    ${PROJECT_SOURCE_DIR}/docs/Doxyfile.in
    ${PROJECT_BINARY_DIR}/Doxyfile
    @ONLY)

  add_custom_target(
    doc
#      ALL
    ${DOXYGEN_EXECUTABLE}
    ${PROJECT_BINARY_DIR}/Doxyfile
    COMMENT "Generating API documentation with Doxygen"
    VERBATIM)
    
endif(DOXYGEN_FOUND)
