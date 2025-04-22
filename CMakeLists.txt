cmake_minimum_required(VERSION 3.22)
project(covariance_geometry_ros)

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

add_compile_options(-Wall -Wextra -Wpedantic -Wconversion -Wdouble-promotion -Wfloat-equal -Wshadow)

find_package(ament_cmake REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(covariance_geometry_cpp REQUIRED)

add_definitions(${EIGEN3_DEFINITIONS})

include_directories(
  include
  ${EIGEN3_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME} SHARED src/covariance_geometry_ros.cpp)
target_include_directories(
  ${PROJECT_NAME}
  PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include>"
)

add_dependencies(
  ${PROJECT_NAME}
  covariance_geometry_cpp
)

ament_target_dependencies(
  ${PROJECT_NAME}
  covariance_geometry_cpp
  Eigen3
  geometry_msgs
  tf2
  tf2_geometry_msgs
)

install(
  TARGETS ${PROJECT_NAME}
  EXPORT export_${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES
  DESTINATION include
)

install(DIRECTORY include/ DESTINATION include)

ament_export_targets(export_${PROJECT_NAME} HAS_LIBRARY_TARGET)

ament_export_dependencies(
  Eigen3
  covariance_geometry_cpp
  geometry_msgs
  tf2
  tf2_geometry_msgs
)

if(BUILD_TESTING)
  set(TEST_TARGETS
    test_composition
    test_conversion
  )

  find_package(ament_lint_auto REQUIRED)
  find_package(ament_cmake_gtest REQUIRED)
  ament_lint_auto_find_test_dependencies()
  ament_find_gtest()

  foreach(test_name ${TEST_TARGETS})
    ament_add_gtest("${test_name}" "test/${test_name}.cpp")
    target_link_libraries(
      "${test_name}"
      ${PROJECT_NAME})
  endforeach()
endif()

ament_package()
