cmake_minimum_required(VERSION 3.22)
project(livox_ros_driver2)

# Define a unique name for the library target
set(LIVOX_DRIVER_LIB_TARGET ${PROJECT_NAME}_lib)

find_package(ament_cmake_auto REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(std_msgs REQUIRED)
find_package(rclcpp REQUIRED)
find_package(pcl_conversions REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(builtin_interfaces REQUIRED)
find_package(livox_sdk2 REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/CustomMsg.msg"
  "msg/CustomPoint.msg"
  DEPENDENCIES
    std_msgs
)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 14)

list(INSERT CMAKE_MODULE_PATH 0 "${PROJECT_SOURCE_DIR}/cmake/modules")

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wno-unused-parameter)
endif()

include(cmake/version.cmake)
message(STATUS "${PROJECT_NAME} version: ${LIVOX_ROS_DRIVER2_VERSION}")

add_definitions(-DBUILDING_ROS2)

ament_auto_find_build_dependencies()

find_package(PkgConfig REQUIRED)
pkg_check_modules(APR apr-1)
if(APR_FOUND)
  message(STATUS "APR include dirs: ${APR_INCLUDE_DIRS}")
  message(STATUS "APR libraries: ${APR_LIBRARIES}")
endif()

ament_auto_add_library(${LIVOX_DRIVER_LIB_TARGET} SHARED
  src/livox_ros_driver2.cpp
  src/lddc.cpp
  src/driver_node.cpp
  src/lds.cpp
  src/lds_lidar.cpp
  src/comm/comm.cpp
  src/comm/ldq.cpp
  src/comm/semaphore.cpp
  src/comm/lidar_imu_data_queue.cpp
  src/comm/cache_index.cpp
  src/comm/pub_handler.cpp
  src/parse_cfg_file/parse_cfg_file.cpp
  src/parse_cfg_file/parse_livox_lidar_cfg.cpp
  src/call_back/lidar_common_callback.cpp
  src/call_back/livox_lidar_callback.cpp
)

target_include_directories(${LIVOX_DRIVER_LIB_TARGET} PUBLIC
  ${PCL_INCLUDE_DIRS}
  ${APR_INCLUDE_DIRS}
  ${livox_sdk2_INCLUDE_DIRS}
  3rdparty
  src
)

target_link_libraries(${LIVOX_DRIVER_LIB_TARGET}
  ${APR_LIBRARIES}
  rosidl_typesupport_cpp::rosidl_typesupport_cpp
  livox_sdk2::sdk_core
)

# Add dependency on generated messages
rosidl_get_typesupport_target(cpp_typesupport_target "${PROJECT_NAME}" "rosidl_typesupport_cpp")
target_link_libraries(${LIVOX_DRIVER_LIB_TARGET} "${cpp_typesupport_target}")

rclcpp_components_register_node(${LIVOX_DRIVER_LIB_TARGET}
  PLUGIN "livox_ros::DriverNode"
  EXECUTABLE ${PROJECT_NAME}_node
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_auto_package(INSTALL_TO_SHARE
  config
  launch
)
