cmake_minimum_required(VERSION 3.14)
project(BGFXCMake)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Makefile generators on apple need this flag to compile mixed objective/c++
if(APPLE AND NOT XCODE)
	set(CMAKE_CXX_FLAGS "-ObjC++")
endif()

option(BX_AMALGAMATED "Amalgamated bx build for faster compilation" OFF)

if(NOT BX_DIR)
	set(BX_DIR "${CMAKE_CURRENT_SOURCE_DIR}/bx" CACHE STRING "Base directory of bx.")
endif()

add_subdirectory(cmake/bx)