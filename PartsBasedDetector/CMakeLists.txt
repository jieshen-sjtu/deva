# PartsBasedDetector CMakeLists.txt
cmake_minimum_required(VERSION 2.8.6)

# -----------------------------------------------
# USER-DEFINED VARIABLES
option(BUILD_EXECUTABLE "Build as executable to test functionality"     ON)
option(BUILD_DOC        "Build documentation with Doxygen"              OFF)
option(USE_OPENMP       "Build with OpenMP support for multithreading"  ON)

# --------------------
# CATKIN
set(BUILD_CATKIN OFF)
if (catkin_FOUND)
    set(BUILD_CATKIN ON)
    set(BUILD_EXECUTABLE OFF)
    project(object_recognition_by_parts)
    catkin_stack()
    catkin_project(object_recognition_by_parts
                   DEPENDS Boost OpenCV cvmatio
                   INCLUDE_DIRS include
    )
    #install targets for all things python
    catkin_python_setup()
    find_package(ecto REQUIRED)
    find_package(object_recognition_core REQUIRED)
    # Set the install module path
    set_ecto_install_package_name(${PROJECT_NAME}/ecto_cells)
    # deal with cvmatio
    find_package(cvmatio QUIET) 
    include_directories(SYSTEM ${cvmatio_INCLUDE_DIRS})
else()
    project(PartsBasedDetector)
    # find cvmatio manually
    include_directories(../cvmatio/include)
    find_library(cvmatio_LIBRARIES cvmatio PATHS ../cvmatio/lib)
endif()


# --------------------
# COMMON

# include all warning types
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Werror -Wno-unused-parameter -pedantic")

# check if the MatlabIO dependency was successfully found
if (cvmatio_LIBRARIES)
    set(cvmatio_FOUND ON)
else()
    set(cvmatio_FOUND OFF)
endif()

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug CACHE STRING
        "Choose the type of build. Options are: None, Debug, Release, RelWithDebInfo, MinSizeRel"
    FORCE)
endif()

# find the dependencies
find_package(Boost COMPONENTS system filesystem REQUIRED)
find_package(OpenCV REQUIRED)

# add OpenMP support
if (USE_OPENMP)
    find_package(OpenMP)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")
endif()

# add vectorization support
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4.1")
set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -msse4.1")

# use highest level of optimization in Release mode
if ("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
    set(CMAKE_C_FLAGS   "${CMAKE_C_FLAGS}   -O3")
endif ()


# include the dependencies
include_directories(SYSTEM  ${OPENCV_INCLUDE_DIRS}
                            ${Boost_INCLUDE_DIRS}
)
include_directories("include")

# add the main executable/library
add_subdirectory(src)

# add documentation
if(BUILD_DOC)
    find_package(Doxygen)
    configure_file(${PROJECT_SOURCE_DIR}/doc/Doxyfile.in ${PROJECT_SOURCE_DIR}/doc/Doxyfile @ONLY)
    add_custom_target(${PROJECT_NAME}_doc ALL ${DOXYGEN_EXECUTABLE} ${PROJECT_SOURCE_DIR}/doc/Doxyfile)
endif()

# add tests
if(BUILD_TEST)
  enable_testing()
  add_subdirectory(test)
endif()

# add ecto cells
if (${ecto_FOUND} AND ${object_recognition_core_FOUND})    
    add_subdirectory(cells)
    add_subdirectory(python)
endif()


# --------------------
# Build summary
message("")
message("---------------------------------------------")
message(${PROJECT} " Configuration Summary")
message("")
message("Building with ECTO/Catkin:     ${BUILD_CATKIN}")
message("Build as executable:           ${BUILD_EXECUTABLE}")
message("Build with documentation:      ${BUILD_DOC}")
message("Build with threading (OpenMP): ${USE_OPENMP}")
message("Build with MatlabIO support:   ${cvmatio_FOUND}")
message("Build type:                    ${CMAKE_BUILD_TYPE}")
message("")
message("---------------------------------------------")
message("")
