cmake_minimum_required(VERSION 3.9)

project(ORB_SLAM3)

set(ORB_SLAM3_VERSION "0.3.0")

set(LIBRARY_NAME ORB_SLAM3)

################################################################################
# Paths
################################################################################

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

if(UNIX)
   include(GNUInstallDirs)

   set(ORB_SLAM3_INC_INSTALL_DIR "/usr/local/include/${PROJECT_NAME}")
   set(ORB_SLAM3_RUNTIME_INSTALL_DIR "${CMAKE_INSTALL_BINDIR}")
   set(ORB_SLAM3_LIBRARY_INSTALL_DIR "/usr/local/lib")
   set(ORB_SLAM3_ARCHIVE_INSTALL_DIR "/usr/local/lib")
   set(ORB_SLAM3_FRAMEWORK_INSTALL_DIR "/usr/local/lib")

   set(ORB_SLAM3_CMAKE_CONFIG_INSTALL_DIR "/usr/local/lib/cmake/ORB_SLAM3")
   set(ORB_SLAM3_ADDITIONAL_FILES_INSTALL_DIR "${CMAKE_INSTALL_DATADIR}/ORB_SLAM3")
endif()

################################################################################
# Include CMake dependencies
################################################################################

include(CheckCXXCompilerFlag)
include(CheckLanguage)
include(CMakePackageConfigHelpers)

################################################################################
# Compiler specific configuration
################################################################################


if(CMAKE_BUILD_TYPE)
   message(STATUS "Build type specified as ${CMAKE_BUILD_TYPE}")
else()
   message(STATUS "Build type not specified, using Release")
   set(CMAKE_BUILD_TYPE Release)
   set(IS_DEBUG OFF)
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

################################################################################
# Flags
################################################################################

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall   -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE}")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE}")

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

################################################################################
# Dependencies
################################################################################

find_package(OpenCV 3)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

MESSAGE("OPENCV VERSION:")
MESSAGE(${OpenCV_VERSION})

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)

################################################################################
# Sources
################################################################################

include_directories(
   ${PROJECT_SOURCE_DIR}
   ${PROJECT_SOURCE_DIR}/include
   ${PROJECT_SOURCE_DIR}/include/CameraModels
   ${EIGEN3_INCLUDE_DIR}
   ${Pangolin_INCLUDE_DIRS}
)

set(HEADERS
        include/System.h
        include/Tracking.h
        include/LocalMapping.h
        include/LoopClosing.h
        include/ORBextractor.h
        include/ORBmatcher.h
        include/ORBVocabulary.h
        include/FrameDrawer.h
        include/Converter.h
        include/MapPoint.h
        include/KeyFrame.h
        include/Atlas.h
        include/Map.h
        include/MapDrawer.h
        include/Optimizer.h
        include/PnPsolver.h
        include/Frame.h
        include/KeyFrameDatabase.h
        include/Sim3Solver.h
        include/Initializer.h
        include/Viewer.h
        include/ImuTypes.h
        include/G2oTypes.h
        include/CameraModels/GeometricCamera.h
        include/CameraModels/Pinhole.h
        include/CameraModels/KannalaBrandt8.h
        include/OptimizableTypes.h
        include/MLPnPsolver.h
        include/TwoViewReconstruction.h
)

set(SOURCES
        src/System.cc
        src/Tracking.cc
        src/LocalMapping.cc
        src/LoopClosing.cc
        src/ORBextractor.cc
        src/ORBmatcher.cc
        src/FrameDrawer.cc
        src/Converter.cc
        src/MapPoint.cc
        src/KeyFrame.cc
        src/Atlas.cc
        src/Map.cc
        src/MapDrawer.cc
        src/Optimizer.cc
        src/PnPsolver.cc
        src/Frame.cc
        src/KeyFrameDatabase.cc
        src/Sim3Solver.cc
        src/Initializer.cc
        src/Viewer.cc
        src/ImuTypes.cc
        src/G2oTypes.cc
        src/CameraModels/Pinhole.cpp
        src/CameraModels/KannalaBrandt8.cpp
        src/OptimizableTypes.cpp
        src/MLPnPsolver.cpp
        src/TwoViewReconstruction.cc
)

add_library(${LIBRARY_NAME} SHARED
        ${HEADERS}
        ${SOURCES}
)

add_subdirectory(Thirdparty/g2o)
add_subdirectory(Thirdparty/DBoW2)

target_include_directories(${LIBRARY_NAME} PUBLIC
     $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}> # for headers when building
     $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include> # for headers when building
     $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/CameraModels> # for headers when building
     $<INSTALL_INTERFACE:${ORB_SLAM3_INC_INSTALL_DIR}> # for client in install mode
)

target_link_libraries(${LIBRARY_NAME}
     PUBLIC
        ORB_SLAM3_DBoW2
        ORB_SLAM3_g2o
        ${OpenCV_LIBS}
        ${EIGEN3_LIBS}
        ${Pangolin_LIBRARIES}
        -lboost_serialization
        -lcrypto
        -lGL
        -lGLU
        -lGLEW
)

add_definitions(
        -Wno-deprecated-declarations
)

## Compile Orb vocabulary
add_executable(bin_vocabulary Vocabulary/bin_vocabulary.cc)
target_link_libraries(bin_vocabulary ${LIBRARY_NAME})

################################################################################
# Create Targets
################################################################################

# Library
install(TARGETS ${LIBRARY_NAME} EXPORT ORB_SLAM3Targets
        RUNTIME       DESTINATION ${ORB_SLAM3_RUNTIME_INSTALL_DIR}
        LIBRARY       DESTINATION ${ORB_SLAM3_LIBRARY_INSTALL_DIR}
        ARCHIVE       DESTINATION ${ORB_SLAM3_ARCHIVE_INSTALL_DIR}
        FRAMEWORK     DESTINATION ${ORB_SLAM3_FRAMEWORK_INSTALL_DIR})

# Headers
install(
        FILES ${HEADERS}
        DESTINATION ${ORB_SLAM3_INC_INSTALL_DIR}
)

################################################################################
# Install All
################################################################################

set_target_properties(${LIBRARY_NAME} PROPERTIES OUTPUT_NAME "ORB_SLAM3-${ORB_SLAM3_VERSION}")

# Create and install version file
set(version_file "${CMAKE_CURRENT_BINARY_DIR}/ORB_SLAM3ConfigVersion.cmake")
write_basic_package_version_file(${version_file}
        VERSION ${ORB_SLAM3_VERSION}
        COMPATIBILITY ExactVersion
        )
install(FILES ${version_file}
        DESTINATION ${ORB_SLAM3_CMAKE_CONFIG_INSTALL_DIR})

# install readme and license
install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE" "${CMAKE_CURRENT_SOURCE_DIR}/README.md"
        DESTINATION ${ORB_SLAM3_ADDITIONAL_FILES_INSTALL_DIR}
        )

install(
        EXPORT ORB_SLAM3Targets
        DESTINATION ${ORB_SLAM3_CMAKE_CONFIG_INSTALL_DIR}
        FILE ORB_SLAM3Config.cmake
)

################################################################################
# Build Examples
################################################################################

if (BUILD_EXAMPLES)
   # Build examples

   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/RGB-D)
   add_executable(rgbd_tum
           Examples/RGB-D/rgbd_tum.cc)
   target_link_libraries(rgbd_tum ${PROJECT_NAME})


   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo)

   add_executable(stereo_kitti
           Examples/Stereo/stereo_kitti.cc)
   target_link_libraries(stereo_kitti ${PROJECT_NAME})

   add_executable(stereo_euroc
           Examples/Stereo/stereo_euroc.cc)
   target_link_libraries(stereo_euroc ${PROJECT_NAME})

   add_executable(stereo_tum_vi
           Examples/Stereo/stereo_tum_vi.cc)
   target_link_libraries(stereo_tum_vi ${PROJECT_NAME})


   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular)

   add_executable(mono_tum
           Examples/Monocular/mono_tum.cc)
   target_link_libraries(mono_tum ${PROJECT_NAME})

   add_executable(mono_kitti
           Examples/Monocular/mono_kitti.cc)
   target_link_libraries(mono_kitti ${PROJECT_NAME})

   add_executable(mono_euroc
           Examples/Monocular/mono_euroc.cc)
   target_link_libraries(mono_euroc ${PROJECT_NAME})

   add_executable(mono_tum_vi
           Examples/Monocular/mono_tum_vi.cc)
   target_link_libraries(mono_tum_vi ${PROJECT_NAME})


   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Monocular-Inertial)

   add_executable(mono_inertial_euroc
           Examples/Monocular-Inertial/mono_inertial_euroc.cc)
   target_link_libraries(mono_inertial_euroc ${PROJECT_NAME})

   add_executable(mono_inertial_tum_vi
           Examples/Monocular-Inertial/mono_inertial_tum_vi.cc)
   target_link_libraries(mono_inertial_tum_vi ${PROJECT_NAME})


   set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/Examples/Stereo-Inertial)

   add_executable(stereo_inertial_euroc
           Examples/Stereo-Inertial/stereo_inertial_euroc.cc)
   target_link_libraries(stereo_inertial_euroc ${PROJECT_NAME})


   add_executable(stereo_inertial_tum_vi
           Examples/Stereo-Inertial/stereo_inertial_tum_vi.cc)
   target_link_libraries(stereo_inertial_tum_vi ${PROJECT_NAME})
endif()