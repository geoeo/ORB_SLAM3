cmake_minimum_required(VERSION 3.9)

set(LIBRARY_NAME DBoW2)

################################################################################
# Flags
################################################################################

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall  -O3 -march=native")

set(HDRS_DBOW2
  DBoW2/BowVector.h
  DBoW2/FORB.h 
  DBoW2/FClass.h       
  DBoW2/FeatureVector.h
  DBoW2/ScoringObject.h   
  DBoW2/TemplatedVocabulary.h)
set(SRCS_DBOW2
  DBoW2/BowVector.cpp
  DBoW2/FORB.cpp      
  DBoW2/FeatureVector.cpp
  DBoW2/ScoringObject.cpp)

set(HDRS_DUTILS
  DUtils/Random.h
  DUtils/Timestamp.h)
set(SRCS_DUTILS
  DUtils/Random.cpp
  DUtils/Timestamp.cpp)

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()


include_directories(${OpenCV_INCLUDE_DIRS})

add_library(${LIBRARY_NAME} SHARED
        ${SRCS_DBOW2}
        ${SRCS_DUTILS}
)

target_link_libraries(${LIBRARY_NAME}
     PUBLIC
        ${OpenCV_LIBS}
        )

target_include_directories(${LIBRARY_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/DBoW2> # for headers when building
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/DUtils> # for headers when building
        $<INSTALL_INTERFACE:${ORB_SLAM3_INC_INSTALL_DIR}/DBoW2> # for client in install mode
        $<INSTALL_INTERFACE:${ORB_SLAM3_INC_INSTALL_DIR}/DUtils> # for client in install mode
)

set_target_properties(${LIBRARY_NAME} PROPERTIES OUTPUT_NAME "${LIBRARY_NAME}-${ORB_SLAM3_VERSION}")

# Library
install(TARGETS ${LIBRARY_NAME} EXPORT ORB_SLAM3Targets
        RUNTIME       DESTINATION ${ORB_SLAM3_RUNTIME_INSTALL_DIR}
        LIBRARY       DESTINATION ${ORB_SLAM3_LIBRARY_INSTALL_DIR}
        ARCHIVE       DESTINATION ${ORB_SLAM3_ARCHIVE_INSTALL_DIR}
        FRAMEWORK     DESTINATION ${ORB_SLAM3_FRAMEWORK_INSTALL_DIR})

# Headers
install(
        DIRECTORY DBoW2 DUtils
        DESTINATION ${ORB_SLAM3_INC_INSTALL_DIR}/Thirdparty/DBoW2
        FILES_MATCHING
        PATTERN "*.h"
        PATTERN "*.hpp"
)