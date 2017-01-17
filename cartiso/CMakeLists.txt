cmake_minimum_required(VERSION 3.3)
project(Cartiso C)

find_package(MPI REQUIRED)
include_directories(SYSTEM ${MPI_C_INCLUDE_PATH})

set(SOURCES iso.c sfc.c cartiso.c ${CMAKE_CURRENT_SOURCE_DIR}/../osn/open-simplex-noise.c)
set(LIBS pthread)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/.. ${CMAKE_CURRENT_SOURCE_DIR}/../osn)

set(ENABLE_PVTI ON CACHE BOOL "ENABLE PVTI")
if(ENABLE_PVTI)
  list(APPEND SOURCES pvti.c)
  add_definitions(-DHAS_PVTI)
endif()

set(ENABLE_PVTP ON CACHE BOOL "ENABLE PVTP")
if(ENABLE_PVTP)
  list(APPEND SOURCES pvtp.c)
  add_definitions(-DHAS_PVTP)
endif()

set(ENABLE_ADIOS ON CACHE BOOL "ENABLE ADIOS")
if(ENABLE_ADIOS)
  set (CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/../CMake)
  find_package(ADIOS REQUIRED)
  include_directories(SYSTEM ${ADIOS_INCLUDE_DIRS})
  find_package(ADIOS)
  list(APPEND SOURCES adiosfull.c adiosiso.c)
  list(APPEND LIBS ${ADIOS_LIBRARIES})
  add_definitions(-DHAS_ADIOS)
endif()

set(ENABLE_HDF5 ON CACHE BOOL "ENABLE HDF5")
if(ENABLE_HDF5)
  find_package(HDF5 REQUIRED)
  include_directories(SYSTEM ${HDF5_INCLUDE_DIRS})
  list(APPEND SOURCES hdf5i.c hdf5p.c)
  list(APPEND LIBS ${HDF5_LIBRARIES})
  add_definitions(-DHAS_HDF5)
endif()

add_executable(cartiso ${SOURCES})

target_link_libraries(cartiso LINK_PRIVATE ${LIBS} ${MPI_LIBRARIES} m pthread)

option(BUILD_TESTING "Build Testing" OFF)
# Setup testing.
if (BUILD_TESTING)
  include(CTest)
  #add_test(NAME CFullExampleTest COMMAND CFullExample ${CMAKE_CURRENT_SOURCE_DIR}/SampleScripts/feslicescript.py)
  #set_tests_properties(CFullExampleTest PROPERTIES LABELS "PARAVIEW;CATALYST")
endif()

