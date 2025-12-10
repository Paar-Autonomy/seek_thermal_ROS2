cmake_minimum_required(VERSION 3.8)
project(seekthermal_camera)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(image_transport REQUIRED)
find_package(OpenCV REQUIRED COMPONENTS imgproc) 
find_package(sensor_msgs REQUIRED)
find_package(camera_info_manager REQUIRED)

find_library(SEEK_THERMAL_LIB 
  NAMES seek_thermal libseek_thermal
  PATHS /usr/local/lib
  REQUIRED
)

add_executable(seekthermal_node src/seekthermal_camera.cpp)
target_link_libraries(seekthermal_node ${SEEK_THERMAL_LIB})

target_include_directories(seekthermal_node
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    )

ament_target_dependencies(seekthermal_node 
  rclcpp 
  std_msgs 
  sensor_msgs 
  camera_info_manager 
  cv_bridge 
  image_transport 
  OpenCV
)

ament_export_include_directories(
  include )

ament_export_dependencies(
  rclcpp 
  sensor_msgs 
  image_transport 
  cv_bridge 
  camera_info_manager 
  std_msgs)

install(TARGETS 
  seekthermal_node
  DESTINATION lib/${PROJECT_NAME}/
)

install(DIRECTORY
  launch
  DESTINATION share/${PROJECT_NAME}/
)

install(DIRECTORY
  include/seekthermal_camera
  DESTINATION include/${PROJECT_NAME}/
)

string(FIND ${CMAKE_INSTALL_PREFIX} "/opt/ros" _PREFIX_INDEX)
if (${_PREFIX_INDEX} EQUAL 0)
  message(STATUS "Building as a Debian package - adding udev rules as installable files")
  install(FILES
    config/99-seekthermal.rules
    DESTINATION /lib/udev/rules.d
  )
else()
  message(STATUS "Building in a user workspace - not adding udev rules")
endif()

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
