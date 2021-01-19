cmake_minimum_required(VERSION 2.8)

SET(CMAKE_CXX_COMPILER "g++")

PROJECT(ipu_2dtg)

# set c++11 standard
if (CMAKE_VERSION VERSION_LESS "3.1")
  if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11")
  endif ()
else ()
  set (CMAKE_CXX_STANDARD 11)
endif ()

find_package(OpenCV REQUIRED)
# find_package(Boost COMPONENTS filesystem system REQUIRED)

include_directories(${OpenCV_INCLUDE_DIRS})
# include_directories(${Boost_INCLUDE_DIRS})
include_directories("/usr/local/ButlerIPU/include")
#include_directories("/data/bkp/spdlog/include")

#################################### ROS additions #############################

include_directories("/opt/ros/kinetic/include")
link_directories("/opt/ros/kinetic/lib")
include_directories("include")




###############################################################################
# Get the current working branch
execute_process(
  COMMAND git rev-parse --abbrev-ref HEAD
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE GIT_BRANCH
  OUTPUT_STRIP_TRAILING_WHITESPACE
)

# Get the latest abbreviated commit hash of the working branch
execute_process(
  COMMAND git log -1 --format=%h
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE GIT_COMMIT_HASH
  OUTPUT_STRIP_TRAILING_WHITESPACE
)


# message(STATUS "Branch: ${GIT_BRANCH}")
# message(STATUS "Commit id: ${GIT_COMMIT_HASH}")

add_definitions(-DVERSION="${GIT_BRANCH}-${GIT_COMMIT_HASH}")

# add CFLAGS
add_definitions("-O3 -Wall -lstdc++ -Wl,-rpath -fpermissive -Wno-unused-variable -Wno-unused-function -Wno-unused-but-set-variable -std=c++11")

# make a list of all source files for ButkerIPUServer
file(GLOB SOURCES "src/*.h" "src/*.cpp")

# make list of all source files for ButlerIPUCalibration
file(GLOB SOURCES_CALIB "calibration/*.h" "calibration/*.cpp")

# library directories
link_directories("/usr/local/ButlerIPU/lib")




IF(${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "aarch64") 
  # aarch64 settings
  message("Compiling for ${CMAKE_SYSTEM_PROCESSOR}")
  add_executable(ButlerIPUServer ${SOURCES} /usr/local/ButlerIPU/lib/libDMatrix_18.o )
  add_executable(ButlerIPUCalibration ${SOURCES_CALIB} /usr/local/ButlerIPU/lib/libDMatrix_18.o)
  
  target_link_libraries(ButlerIPUServer
    # opencv dependencies
    # ${OpenCV_LIBS}
    opencv_core
    opencv_highgui
    opencv_imgcodecs
    opencv_imgproc
    opencv_calib3d
    opencv_ccalib

    # boost
    # ${Boost_FILESYSTEM_LIBRARY}
    # ${Boost_SYSTEM_LIBRARY}
    boost_filesystem
    boost_system

    # others
    pthread
    m
    ueye_api
    jpeg
    config
    dl
    
    # ROS libraries
    roscpp
    rostime
    rosconsole
    roscpp_serialization
  )

  target_link_libraries(ButlerIPUCalibration
    # opencv dependencies
    # ${OpenCV_LIBS}
    opencv_core
    opencv_highgui
    opencv_imgcodecs
    opencv_imgproc
    opencv_calib3d
    opencv_ccalib

    # boost
    # ${Boost_FILESYSTEM_LIBRARY}
    # ${Boost_SYSTEM_LIBRARY}
    boost_filesystem
    boost_system

    # others
    pthread
    m
    ueye_api
    jpeg
    config
  )

ELSE(${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "aarch64")
  # x86_64 settings
  message("Compiling for ${CMAKE_SYSTEM_PROCESSOR}")
  add_executable(ButlerIPUServer ${SOURCES})
  add_executable(ButlerIPUCalibration ${SOURCES_CALIB})
  target_link_libraries(ButlerIPUServer
    # opencv dependencies
    # ${OpenCV_LIBS}
    opencv_core
    opencv_highgui
    opencv_imgcodecs
    opencv_imgproc
    opencv_calib3d
    opencv_ccalib

    # boost
    # ${Boost_FILESYSTEM_LIBRARY}
    # ${Boost_SYSTEM_LIBRARY}
    boost_filesystem
    boost_system

    # others
    pthread
    m
    ueye_api
    jpeg
    config
    dl

    # Barcode Decoding
    # DMatrix
    DMatrix_18
    dmtx
    barcodedecode

        
    # ROS libraries
    roscpp
    rostime
    rosconsole
    roscpp_serialization
  )

  target_link_libraries(ButlerIPUCalibration
    # opencv dependencies
    # ${OpenCV_LIBS}
    opencv_core
    opencv_highgui
    opencv_imgcodecs
    opencv_imgproc
    opencv_calib3d
    opencv_ccalib

    # boost
    # ${Boost_FILESYSTEM_LIBRARY}
    # ${Boost_SYSTEM_LIBRARY}
    boost_filesystem
    boost_system

    # others
    pthread
    m
    ueye_api
    # DMatrix
    DMatrix_18
    jpeg
    config
  )
ENDIF(${CMAKE_HOST_SYSTEM_PROCESSOR} MATCHES "aarch64")


target_include_directories(ButlerIPUServer PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
)

target_include_directories(ButlerIPUCalibration PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/calibration>
)


