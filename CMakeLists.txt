cmake_minimum_required(VERSION 3.22)
project(DBoW2)

set(CMAKE_CXX_STANDARD 11)

include(CMakeDependentOption)

option(BUILD_DBoW2   "Build DBoW2"            ON)
cmake_dependent_option(BUILD_Demo "Build demo application" ON BUILD_DBoW2 OFF)


if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()

if(MSVC)
  add_compile_options(/W4)
else()
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(HDRS
  include/DBoW2/BowVector.h           include/DBoW2/FBrief.h
  include/DBoW2/QueryResults.h        include/DBoW2/TemplatedDatabase.h   include/DBoW2/FORB.h
  include/DBoW2/DBoW2.h               include/DBoW2/FClass.h              include/DBoW2/FeatureVector.h
  include/DBoW2/ScoringObject.h       include/DBoW2/TemplatedVocabulary.h)
set(SRCS
  src/BowVector.cpp     src/FBrief.cpp        src/FORB.cpp
  src/FeatureVector.cpp src/QueryResults.cpp  src/ScoringObject.cpp)

if(BUILD_DEMO)
  set(OpenCV_EXTRA_COMPONENTS features2d videoio)
endif()

find_package(OpenCV REQUIRED COMPONENTS core ${OpenCV_EXTRA_COMPONENTS})
find_package(Boost REQUIRED COMPONENTS serialization)

include(GNUInstallDirs)

if(BUILD_DBoW2)
  set(LIB_SHARED "SHARED")
  if(WIN32)
    set(LIB_SHARED "STATIC")
  endif(WIN32)
  add_library(${PROJECT_NAME} ${LIB_SHARED} ${SRCS})
  target_include_directories(${PROJECT_NAME} PUBLIC
    ${OpenCV_INCLUDE_DIRS}
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/DBoW2>"
    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/DBoW2>")
  target_link_libraries(${PROJECT_NAME} PUBLIC opencv_core Boost::serialization)
  set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 11)
endif(BUILD_DBoW2)

if(BUILD_Demo)
  add_executable(demo demo/demo.cpp)
  target_link_libraries(demo ${PROJECT_NAME} opencv_videoio opencv_features2d)
  file(COPY demo/images DESTINATION ${CMAKE_BINARY_DIR}/)
endif(BUILD_Demo)

configure_file(src/DBoW2.cmake.in
  "${PROJECT_BINARY_DIR}/DBoW2Config.cmake" @ONLY)

if(BUILD_DBoW2)
  install(TARGETS ${PROJECT_NAME}
    EXPORT DBoW2Targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
  install(DIRECTORY include/DBoW2 DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
  install(EXPORT DBoW2Targets
          FILE DBoW2Targets.cmake
          NAMESPACE DBoW2::
          DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DBoW2)
  include(CMakePackageConfigHelpers)
  configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/src/DBoW2.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/DBoW2Config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/DBoW2)
endif()

install(FILES LICENSE.txt
  DESTINATION ${CMAKE_INSTALL_DOCDIR})
