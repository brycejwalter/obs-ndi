cmake_minimum_required(VERSION 3.5)
project(obs-ndi)

include(external/FindLibObs.cmake)
find_package(LibObs REQUIRED)
find_package(w32-pthreads REQUIRED)

set(OBS_FRONTEND_LIB "OBS_FRONTEND_LIB-NOTFOUND" CACHE FILEPATH "OBS frontend library")
if(OBS_FRONTEND_LIB EQUAL "OBS_FRONTEND_LIB-NOTFOUND")
	message(FATAL_ERROR "Could not find OBS Frontend API\'s library !")
endif()

set(NDISDK_DIR "NDISDK_DIR-NOTFOUND" CACHE PATH "NewTek NDI SDK")
if(NDISDK_DIR EQUAL "NDISDK_DIR-NOTFOUND")
	message(FATAL_ERROR "Could not find NewTek's NDI SDK !")
endif()

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
	set(ARCH 64)
	set(ARCH_NAME "x64")
else()
	set(ARCH 32)
	set(ARCH_NAME "x86")
endif()

set(obs-ndi_SOURCES 
	obs-ndi.cpp
	obs-ndi-source.cpp
	obs-ndi-output.cpp)

set(obs-ndi_HEADERS
	obs-ndi.h)

add_library(obs-ndi MODULE 
	${obs-ndi_SOURCES}
	${obs-ndi_HEADERS})
include_directories( 
	"${LIBOBS_INCLUDE_DIR}/../UI/obs-frontend-api")
target_link_libraries(obs-ndi 
	libobs
	"${OBS_FRONTEND_LIB}")

# Windows
if(WIN32)
	include_directories(
		"${NDISDK_DIR}/Include")
	target_link_libraries(obs-ndi
		w32-pthreads
		"${NDISDK_DIR}/Lib/${ARCH_NAME}/Processing.NDI.Lib.${ARCH_NAME}.lib")
endif()

# OSX (WARNING : needs testing)
if(APPLE)
	include_directories(
		"${NDISDK_DIR}/Non-Windows/OSX/include")
	target_link_libraries(obs-ndi
		"${NDISDK_DIR}/Non-Windows/OSX/bin/x64/libndi.dylib")
endif()

# Linux (WARNING : needs testing)
if(UNIX AND NOT APPLE)
	include_directories(
		"${NDISDK_DIR}/Non-Windows/Linux/include")
	target_link_libraries(obs-ndi
		"${NDISDK_DIR}/Non-Windows/Linux/lib/${ARCH_NAME}/libndi.a")
endif()