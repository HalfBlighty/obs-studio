cmake_minimum_required(VERSION 2.8)
project(obs-python)

set(Python_ADDITIONAL_VERSIONS 3.4)
find_package(PythonLibs 3.4)
find_package(SWIG 2)

if(NOT PYTHONLIBS_FOUND)
	message(STATUS "Python 3 not found, Python plugin disabled")
	return()
endif()

if(NOT SWIG_FOUND)
	message(STATUS "SWIG not found, Python plugin disabled")
	return()
endif()

include(${SWIG_USE_FILE})

include_directories(${PYTHON_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(obs-python_SOURCES
    obs-python-module.c
    py-source.h
    py-obs.h)

add_library(obs-python SHARED
	${obs-python_SOURCES})

string(REGEX REPLACE "_d" "" PYTHON_LIBRARIES "${PYTHON_LIBRARIES}")

target_link_libraries(obs-python
	libobs
	${PYTHON_LIBRARIES})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")

add_definitions(-DSWIG_TYPE_TABLE=obspython -DSWIG_PYTHON_INTERPRETER_NO_DEBUG)
add_subdirectory(swig)

set(SWIG_RUNTIME swig/swigpyrun.h)
add_custom_command(OUTPUT ${SWIG_RUNTIME} 
	WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR} 
	COMMAND ${SWIG_EXECUTABLE} -python -external-runtime ${SWIG_RUNTIME} 
	)
set_source_files_properties(${SWIG_RUNTIME} PROPERTIES GENERATED TRUE) 
add_custom_target(swig-runtime 
	DEPENDS ${SWIG_RUNTIME}
	)

add_dependencies(obs-python swig-runtime)

install_obs_plugin_with_data(obs-python data)
