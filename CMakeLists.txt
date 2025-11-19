# Check cmake version meets our requirements
cmake_minimum_required(VERSION 3.12)

#-----------------------------------------------------------------------
# Enforce an out-of-source build.
#
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message(STATUS "Cimrman requires an out-of-source build.")
  message(STATUS "Please remove these files from ${CMAKE_BINARY_DIR} first:")
  message(STATUS "  CMakeCache.txt")
  message(STATUS "  CMakeFiles")
  message(STATUS "Once these files are removed, create a separate directory")
  message(STATUS "and run CMake from there, pointing it to:")
  message(STATUS "  ${CMAKE_SOURCE_DIR}")
  message(FATAL_ERROR "in-source build detected")
endif()

# Declare project, which will configure compiler for us
project(FalaiseCimrmanPlugin VERSION 1.0.0)

option(CIMRMAN_INPLACE_INSTALL "Install the Cimrman module in the Falaise installation tree" ON)

# Finds Falaise packages and defines cmake variables
find_package(Falaise REQUIRED)
message( STATUS "Found Falaise : ${Falaise_FOUND}")
message( STATUS "- Falaise version : ${Falaise_VERSION}")

if (CIMRMAN_INPLACE_INSTALL)
  message( STATUS "- Falaise_CMAKE_CONFIG_DIR: '${Falaise_CMAKE_CONFIG_DIR}'")
  set(FalaiseModuleInstallDir "${Falaise_CMAKE_CONFIG_DIR}/../../Falaise/modules")
  message( STATUS "- Falaise plug-in dir : '${FalaiseModuleInstallDir}'")
  set(CMAKE_INSTALL_PLUGINDIR "${FalaiseModuleInstallDir}")
else()
  set(CMAKE_INSTALL_PLUGINDIR "${CMAKE_INSTALL_PREFIX}/lib/Falaise/modules")
endif()
    
  
# Finds ROOT
find_package(ROOT REQUIRED)
message( STATUS "Found ROOT : ${ROOT_FOUND}")
message( STATUS "- ROOT version : ${ROOT_VERSION}")
message( STATUS "- ROOT libraries : ${ROOT_LIBRARIES}")

include_directories(include ${Bayeux_INCLUDE_DIRS})

include_directories(${CMAKE_CURRENT_BINARY_DIR}/source)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/source)

list(APPEND Cimrman_HEADERS
  source/tkrec/Visu.h
  source/tkrec/Algos.h
  source/tkrec/Geometry.h
  source/tkrec/Cluster.h
  source/tkrec/DelayedCluster.h
  source/tkrec/Event.h
  source/tkrec/OMHit.h
  source/tkrec/Point.h
  source/tkrec/Association.h
  source/tkrec/Cimrman.h
  source/tkrec/Track.h
  source/tkrec/Trajectory.h
  source/tkrec/TrackerHit.h
  source/tkrec/Solution.h
  source/tkrec/PreclusterSolution.h
  source/tkrec/Precluster.h
  source/tkrec/LinearFit.h
  source/tkrec/Likelihood.h
  )

list(APPEND Cimrman_SOURCES
  source/tkrec/Cimrman.cpp
  source/tkrec/Visu.cpp
  source/tkrec/Algos.cpp
  source/tkrec/Event.cpp
  source/tkrec/Trajectory.cpp
  source/tkrec/Track.cpp
  source/tkrec/Cluster.cpp
  source/tkrec/DelayedCluster.cpp
  source/tkrec/OMHit.cpp
  source/tkrec/TrackerHit.cpp
  source/tkrec/Point.cpp
  source/tkrec/Association.cpp
  source/tkrec/Solution.cpp
  source/tkrec/PreclusterSolution.cpp
  source/tkrec/Precluster.cpp
  source/tkrec/LinearFit.cpp
  source/tkrec/Likelihood.cpp
  )

add_library(Falaise_Cimrman SHARED
  ${Cimrman_SOURCES}
  ${Cimrman_HEADERS} )

target_include_directories(Falaise_Cimrman PUBLIC Falaise::Falaise PUBLIC ROOT::Root)

target_link_libraries(Falaise_Cimrman Falaise::Falaise ${ROOT_LIBRARIES} ROOT::Geom)
# PUBLIC Falaise::Falaise FalaiseModule ${ROOT_LIBRARIES} ROOT::Geom)  

set_target_properties(Falaise_Cimrman PROPERTIES INSTALL_RPATH_USE_LINK_PATH 1)

# Apple linker requires dynamic lookup of symbols, so we
# add link flags on this platform
if(APPLE)
  set_target_properties(Falaise_Cimrman
    PROPERTIES
    LINK_FLAGS "-undefined dynamic_lookup"
    INSTALL_RPATH "@loader_path"
  )
endif()


# Install Targets
message( STATUS "CMAKE_INSTALL_PLUGINDIR = '${CMAKE_INSTALL_PLUGINDIR}' ")
install(TARGETS Falaise_Cimrman DESTINATION ${CMAKE_INSTALL_PLUGINDIR})

# Test support:
option(FalaiseCimrmanPlugin_ENABLE_TESTING "Build unit testing system for FalaiseCimrmanPlugin" OFF)
if(FalaiseCimrmanPlugin_ENABLE_TESTING)
 enable_testing()
 add_subdirectory(testing)
endif()

# configure_file("CimrmanPipeline.conf.in" "CimrmanPipeline.conf" @ONLY)
