cmake_minimum_required(VERSION 3.14)

project(g2o)

set(LIBRARY_NAME ORB_SLAM3_g2o)

if(CMAKE_BUILD_TYPE)
  message(STATUS "Build type specified as ${CMAKE_BUILD_TYPE}")
else()
  message(STATUS "Build type not specified, using Release")
  set(CMAKE_BUILD_TYPE Release)
  set(IS_DEBUG OFF)
endif()

set(CMAKE_CXX_STANDARD 11)

################################################################################
# Flags
################################################################################

set(CMAKE_LEGACY_CYGWIN_WIN32 0)

set(g2o_C_FLAGS)
set(g2o_CXX_FLAGS)

# There seems to be an issue with MSVC8
# see http://eigen.tuxfamily.org/bz/show_bug.cgi?id=83
if(MSVC90)
  add_definitions(-DEIGEN_DONT_ALIGN_STATICALLY=1)
  message(STATUS "Disabling memory alignment for MSVC8")
endif(MSVC90)

# Detect OS and define macros appropriately
if(UNIX)
  add_definitions(-DUNIX)
  message(STATUS "Compiling on Unix")
endif(UNIX)

# Eigen library parallelise itself, though, presumably due to performance issues
# OPENMP is experimental. We experienced some slowdown with it
find_package(OpenMP)
set(G2O_USE_OPENMP OFF CACHE BOOL "Build g2o with OpenMP support (EXPERIMENTAL)")
if(OPENMP_FOUND AND G2O_USE_OPENMP)
  set (G2O_OPENMP 1)
  set(g2o_C_FLAGS "${g2o_C_FLAGS} ${OpenMP_C_FLAGS}")
  set(g2o_CXX_FLAGS "${g2o_CXX_FLAGS} -DEIGEN_DONT_PARALLELIZE ${OpenMP_CXX_FLAGS}")
  message(STATUS "Compiling with OpenMP support")
endif(OPENMP_FOUND AND G2O_USE_OPENMP)

# Compiler specific options for gcc
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O3")

# activate warnings !!!
set(g2o_C_FLAGS "${g2o_C_FLAGS} -Wall -W")
set(g2o_CXX_FLAGS "${g2o_CXX_FLAGS} -Wall -W")

# specifying compiler flags
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${g2o_CXX_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${g2o_C_FLAGS}")

# Find Eigen3
set(EIGEN3_INCLUDE_DIR ${G2O_EIGEN3_INCLUDE})
find_package(Eigen3 3.1.0 REQUIRED)
if(EIGEN3_FOUND)
  set(G2O_EIGEN3_INCLUDE ${EIGEN3_INCLUDE_DIR} CACHE PATH "Directory of Eigen3")
else(EIGEN3_FOUND)
  set(G2O_EIGEN3_INCLUDE "" CACHE PATH "Directory of Eigen3")
endif(EIGEN3_FOUND)

# Generate config.h
set(G2O_CXX_COMPILER "${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER}")
configure_file(config.h.in "${PROJECT_BINARY_DIR}/g2o/config.h")

# Set up the top-level include directories
include_directories(
  g2o/core
  g2o/types
  g2o/stuff
  g2o/solvers
  ${G2O_EIGEN3_INCLUDE}
)

# Include the subdirectories
add_library(${LIBRARY_NAME} SHARED
  #types
  g2o/types/types_sba.h
  g2o/types/types_six_dof_expmap.h
  g2o/types/types_sba.cpp
  g2o/types/types_six_dof_expmap.cpp
  g2o/types/types_seven_dof_expmap.cpp
  g2o/types/types_seven_dof_expmap.h
  g2o/types/se3quat.h
  g2o/types/se3_ops.h
  g2o/types/se3_ops.hpp
  #core
  g2o/core/base_edge.h
  g2o/core/base_binary_edge.h
  g2o/core/hyper_graph_action.cpp
  g2o/core/base_binary_edge.hpp
  g2o/core/hyper_graph_action.h
  g2o/core/base_multi_edge.h
  g2o/core/hyper_graph.cpp
  g2o/core/base_multi_edge.hpp
  g2o/core/hyper_graph.h
  g2o/core/base_unary_edge.h
  g2o/core/linear_solver.h
  g2o/core/base_unary_edge.hpp
  g2o/core/marginal_covariance_cholesky.cpp
  g2o/core/base_vertex.h
  g2o/core/marginal_covariance_cholesky.h
  g2o/core/base_vertex.hpp
  g2o/core/matrix_structure.cpp
  g2o/core/batch_stats.cpp
  g2o/core/matrix_structure.h
  g2o/core/batch_stats.h
  g2o/core/openmp_mutex.h
  g2o/core/block_solver.h
  g2o/core/block_solver.hpp
  g2o/core/parameter.cpp
  g2o/core/parameter.h
  g2o/core/cache.cpp
  g2o/core/cache.h
  g2o/core/optimizable_graph.cpp
  g2o/core/optimizable_graph.h
  g2o/core/solver.cpp
  g2o/core/solver.h
  g2o/core/creators.h
  g2o/core/optimization_algorithm_factory.cpp
  g2o/core/estimate_propagator.cpp
  g2o/core/optimization_algorithm_factory.h
  g2o/core/estimate_propagator.h
  g2o/core/factory.cpp
  g2o/core/optimization_algorithm_property.h
  g2o/core/factory.h
  g2o/core/sparse_block_matrix.h
  g2o/core/sparse_optimizer.cpp
  g2o/core/sparse_block_matrix.hpp
  g2o/core/sparse_optimizer.h
  g2o/core/hyper_dijkstra.cpp
  g2o/core/hyper_dijkstra.h
  g2o/core/parameter_container.cpp
  g2o/core/parameter_container.h
  g2o/core/optimization_algorithm.cpp
  g2o/core/optimization_algorithm.h
  g2o/core/optimization_algorithm_with_hessian.cpp
  g2o/core/optimization_algorithm_with_hessian.h
  g2o/core/optimization_algorithm_levenberg.cpp
  g2o/core/optimization_algorithm_levenberg.h
  g2o/core/optimization_algorithm_gauss_newton.cpp
  g2o/core/optimization_algorithm_gauss_newton.h
  g2o/core/jacobian_workspace.cpp
  g2o/core/jacobian_workspace.h
  g2o/core/robust_kernel.cpp
  g2o/core/robust_kernel.h
  g2o/core/robust_kernel_factory.cpp
  g2o/core/robust_kernel_factory.h
  g2o/core/robust_kernel_impl.cpp
  g2o/core/robust_kernel_impl.h
  #stuff
  g2o/stuff/string_tools.h
  g2o/stuff/color_macros.h
  g2o/stuff/macros.h
  g2o/stuff/timeutil.cpp
  g2o/stuff/misc.h
  g2o/stuff/timeutil.h
  g2o/stuff/os_specific.c
  g2o/stuff/os_specific.h
  g2o/stuff/string_tools.cpp
  g2o/stuff/property.cpp
  g2o/stuff/property.h
)

target_include_directories(${LIBRARY_NAME} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/g2o/core> # for headers when building
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/g2o/types> # for headers when building
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/g2o/solvers> # for headers when building
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/g2o/stuff> # for headers when building
        $<INSTALL_INTERFACE:${ORB_SLAM3_INC_INSTALL_DIR}/Thirdparty/g2o/g2o/core> # for client in install mode
        $<INSTALL_INTERFACE:${ORB_SLAM3_INC_INSTALL_DIR}/Thirdparty/g2o/g2o/types> # for client in install mode
        $<INSTALL_INTERFACE:${ORB_SLAM3_INC_INSTALL_DIR}/Thirdparty/g2o/g2o/solvers> # for client in install mode
        $<INSTALL_INTERFACE:${ORB_SLAM3_INC_INSTALL_DIR}/Thirdparty/g2o/g2o/stuff> # for client in install mode
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
        DIRECTORY g2o
        DESTINATION ${ORB_SLAM3_INC_INSTALL_DIR}/Thirdparty/g2o
        FILES_MATCHING
        PATTERN "*.h"
        PATTERN "*.hpp"
        PATTERN "*.in"
)

install(
        FILES ${PROJECT_BINARY_DIR}/g2o/config.h config.h.in
        DESTINATION ${ORB_SLAM3_INC_INSTALL_DIR}/Thirdparty/g2o
)
