SET(PROJECT_NAME Sophus)

PROJECT(${PROJECT_NAME})
CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

SET( CMAKE_VERBOSE_MAKEFILE ON)

################################################################################
# Release by default
# Turn on Debug with "-DCMAKE_BUILD_TYPE=Debug"
IF( NOT CMAKE_BUILD_TYPE )
   SET( CMAKE_BUILD_TYPE Release )
ENDIF()

IF( CMAKE_COMPILER_IS_GNUCXX )
   SET(CMAKE_CXX_FLAGS_DEBUG  "-O0 -g")
   SET(CMAKE_CXX_FLAGS_RELEASE "-O3")

   ADD_DEFINITIONS("-Wall -Werror")
ENDIF()

################################################################################
# Add local path for finding packages, set the local version first
SET(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake_modules" ${CMAKE_MODULE_PATH})
include( install_package ) 

################################################################################
# Create variables used for exporting (from build tree) in SophusConfig.cmake
SET( Sophus_LIBRARIES "" )
SET( Sophus_INCLUDE_DIR ${PROJECT_SOURCE_DIR} )

################################################################################
FIND_PACKAGE( Eigen3 REQUIRED )
LIST( APPEND Sophus_INCLUDE_DIR ${EIGEN3_INCLUDE_DIR} )

################################################################################
SET( SOURCE_DIR "sophus")

SET( TEMPLATES so2
               se2
               so3
               se3
               rxso3
               sim3
)

SET( SOURCES ${SOURCE_DIR}/sophus.hpp ${SOURCE_DIR}/ensure.hpp
             ${SOURCE_DIR}/example_ensure_handler.cpp )

FOREACH(templ ${TEMPLATES})
  LIST(APPEND SOURCES ${SOURCE_DIR}/${templ}.hpp)
ENDFOREACH(templ)

# Add sources as custom target so that they are shown in IDE's
ADD_CUSTOM_TARGET( libsophus SOURCES ${SOURCES} )

################################################################################
# Create 'test' make target using ctest
option(BUILD_TESTS "Build tests." ON)
if(BUILD_TESTS)
    ENABLE_TESTING()
    ADD_SUBDIRECTORY( test )
endif()

#######################################################
## Generate Doxygen documentation target (make doc)
find_package(Doxygen)
if(DOXYGEN_FOUND)
    configure_file(${CMAKE_CURRENT_SOURCE_DIR}/doxygen/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
    file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc )
    add_custom_target(sophus_doc
        ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc
        COMMENT "Generating API documentation with Doxygen" VERBATIM
    )
endif()

################################################################################
OPTION( EXPORT_Sophus
  "Should the Sophus package be exported for use by other software" OFF )

# Export package for use from the build tree
IF( EXPORT_Sophus )
  EXPORT( PACKAGE Sophus )
ENDIF()

# Create the SophusConfig.cmake file for other cmake projects.
# ... for the build tree
SET( CONFIG_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
SET( CONFIG_DIR ${CMAKE_CURRENT_BINARY_DIR})
CONFIGURE_FILE( ${CMAKE_CURRENT_SOURCE_DIR}/SophusConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/SophusConfig.cmake @ONLY )

# install files and create FindSophus.cmake and Sophus.pc pkg-config files
install_package(
    PKG_NAME Sophus
    VERSION 0.9
    DESCRIPTION "C++ Lie Algebra library based on Eigen."
    INSTALL_HEADERS sophus/so2.hpp sophus/se2.hpp sophus/so3.hpp sophus/se3.hpp sophus/rxso3.hpp sophus/sim3.hpp
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/sophus
    INCLUDE_DIRS ${CMAKE_INSTALL_PREFIX}/include/sophus ${EIGEN3_INCLUDE_DIR}
    )

