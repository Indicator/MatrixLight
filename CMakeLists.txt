#--------------------------------------------------------------
# MatrixLight
#--------------------------------------------------------------
cmake_minimum_required(VERSION 2.4)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/)
project(MatrixLight)
set (MatrixLight_VERSION_MAJOR 2)
set (MatrixLight_VERSION_MINOR 0)
set (MatrixLight_VERSION_PATCH 2)

# Use a low version compiler to compile library, in order to make it compatible with
# application want to be compiled with latest compiler.
set(CMAKE_PREFIX_PATH 
    /share/apps/matlab
  /home/zywang/work/allbio/contrib/hdf5/
  /opt/cray/hdf5/1.8.11/CRAY/81/ 
  /home/zywang/program/hdf5-1.8.10-linux-x86_64-static/
  /home/zywang/program/hdf5-1.8.10-linux-x86_64-shared/
)

set (CMAKE_CXX_FLAGS "-O3 ")

#--------------------------------------------------------------
# global headers
#--------------------------------------------------------------
file(GLOB_RECURSE headers include/*.hxx)
include_directories(include)

set(Gsl_INCLUDE_DIRS $ENV{GSL_DIR}/include)
set(Gsl_LIBRARY_DIRS $ENV{GSL_DIR}/lib)
FIND_PACKAGE(GSL REQUIRED)
include_directories(${GSL_INCLUDE_DIRS} ${GSLCBLAS_INCLUDE_DIRS} $ENV{GSL_DIR}/include)
set(LIBS ${LIBS} ${GSL_LIBRARIES} ${GSLCBLAS_LIBRARIES} $ENV{GSL_DIR}/lib)
link_directories(${LIBS})

if(COMMAND cmake_policy)
   cmake_policy(SET CMP0003 NEW)
endif(COMMAND cmake_policy)
#--------------------------------------------------------------
# source directory
#--------------------------------------------------------------
#add_subdirectory(src)
add_library (MatrixLight matrix.cpp) 
#--------------------------------------------------------------
# options
#--------------------------------------------------------------

#OPTION(WITH_HDF5 "Include HDF5" ON)

#set(HDF5 
#--------------------------------------------------------------
# HDF5
#--------------------------------------------------------------
find_package(HDF5)
if(HDF5_FOUND)
   message(STATUS "Build with HDF5 support")
   set(HDF5_LIBS hdf5 hdf5_hl hdf5_cpp hdf5_hl_cpp)
   include_directories(${HDF5_INCLUDE_DIR})
   add_definitions(${HDF5_CPPFLAGS})
   add_definitions(-DH5_SUPPORT)
   link_directories(${HDF5_LIBRARY_DIRS})
else()
   message(STATUS "build without HDF5")
endif()

#Find matlab engine
#include_directories("/share/apps/matlab/extern/include")
find_package(MATLAB)
if(MATLAB_FOUND)
   message(STATUS "Build with MATLAB ENGINE support")
    include_directories(${MATLAB_INCLUDE_DIR})
    link_directories(${MATLAB_LIBRARY_DIR})

    add_definitions(-DMATLAB_ENGINE)
endif()

#--------------------------------------------------------------
# testing with CTest
#--------------------------------------------------------------
#add_executable (matrix_test matrix_test.cpp)
#target_link_libraries (matrix_test MatrixLight gsl gslcblas ${HDF5_LIBS} ${MATLAB_LIBRARY}) 
#add_test(matrixtest matrix_test)

#if(MATLAB_FOUND)
#add_executable (matrix_test_matlab matrix_test.cpp)
#target_link_libraries (matrix_test_matlab MatrixLight gsl gslcblas ${HDF5_LIBS} ${MATLAB_LIBRARY}) 
#add_test(matrixtest matrix_test_matlab)
#endif()


#--------------------------------------------------------------
# doxygen documentation
#--------------------------------------------------------------
find_package(Doxygen)
if(DOXYGEN_FOUND)
   configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
   add_custom_target(doc ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
      WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
      COMMENT "Generating API documentation with Doxygen" VERBATIM)
endif()




#--------------------------------------------------------------
# install
#--------------------------------------------------------------

install(TARGETS MatrixLight DESTINATION ${CMAKE_CURRENT_SOURCE_DIR} )

