# judge which cmake codes to use 
if(ROS_EDITION STREQUAL "ROS1")

  # Copyright(c) 2019 livoxtech limited.

  cmake_minimum_required(VERSION 3.0)

  include(cmake/version.cmake)
  project(livox_ros_driver2 VERSION ${LIVOX_ROS_DRIVER2_VERSION} LANGUAGES CXX)
  message(STATUS "livox_ros_driver2 version: ${LIVOX_ROS_DRIVER2_VERSION}")

  add_definitions(-DBUILDING_ROS1)

  find_package(Boost 1.54 REQUIRED COMPONENTS system thread chrono)

  find_package(catkin REQUIRED COMPONENTS
    roscpp
    rospy
    sensor_msgs
    std_msgs
    message_generation
    rosbag
    pcl_ros
  )

  find_package(PCL REQUIRED)

  add_message_files(FILES
    CustomPoint.msg
    CustomMsg.msg
  )

  generate_messages(DEPENDENCIES std_msgs)

  find_package(PkgConfig)
  pkg_check_modules(APR apr-1)
  if(APR_FOUND)
    message(${APR_INCLUDE_DIRS})
    message(${APR_LIBRARIES})
  endif()

  catkin_package(CATKIN_DEPENDS roscpp rospy std_msgs message_runtime pcl_ros)

  if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose Release or Debug" FORCE)
  endif()

  set(CMAKE_CXX_STANDARD 14)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
  set(CMAKE_CXX_EXTENSIONS OFF)

  find_library(LIVOX_LIDAR_SDK_LIBRARY liblivox_lidar_sdk_static.a /usr/local/lib)

  link_directories(${PCL_LIBRARY_DIRS})
  add_definitions(${PCL_DEFINITIONS})

  add_executable(${PROJECT_NAME}_node "")

  target_compile_options(${PROJECT_NAME}_node PRIVATE $<$<CXX_COMPILER_ID:GNU>:-Wall>)

  add_dependencies(${PROJECT_NAME}_node ${PROJECT_NAME}_generate_messages_cpp)

  target_sources(${PROJECT_NAME}_node PRIVATE
    src/driver_node.cpp
    src/lds.cpp
    src/lds_lidar.cpp
    src/lddc.cpp
    src/livox_ros_driver2.cpp
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

  target_include_directories(${PROJECT_NAME}_node PUBLIC
    ${catkin_INCLUDE_DIRS}
    ${PCL_INCLUDE_DIRS}
    ${APR_INCLUDE_DIRS}
    3rdparty
    src
  )

  target_link_libraries(${PROJECT_NAME}_node
    ${LIVOX_LIDAR_SDK_LIBRARY}
    ${Boost_LIBRARY}
    ${catkin_LIBRARIES}
    ${PCL_LIBRARIES}
    ${APR_LIBRARIES}
  )

  install(TARGETS ${PROJECT_NAME}_node
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )

  install(DIRECTORY launch_ROS1/
    DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch_ROS1
  )

else(ROS_EDITION STREQUAL "ROS2")
cmake_minimum_required(VERSION 3.14)
project(livox_ros_driver2)

find_package(ament_cmake_auto REQUIRED)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 14)

list(INSERT CMAKE_MODULE_PATH 0 "${PROJECT_SOURCE_DIR}/cmake/modules")

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wno-unused-parameter)
endif()

include(cmake/version.cmake)
project(${PROJECT_NAME} VERSION ${LIVOX_ROS_DRIVER2_VERSION} LANGUAGES CXX)
message(STATUS "${PROJECT_NAME} version: ${LIVOX_ROS_DRIVER2_VERSION}")

add_definitions(-DBUILDING_ROS2)

ament_auto_find_build_dependencies(REQUIRED
  rclcpp
  pcl_conversions
  sensor_msgs
  std_msgs
  builtin_interfaces
  rosidl_default_generators
  livox_interfaces
  PCL
)

find_package(PkgConfig REQUIRED)
pkg_check_modules(APR apr-1)
if(APR_FOUND)
  message(STATUS "APR include dirs: ${APR_INCLUDE_DIRS}")
  message(STATUS "APR libraries: ${APR_LIBRARIES}")
endif()

find_library(LIVOX_LIDAR_SDK_LIBRARY liblivox_lidar_sdk_shared.so /usr/local/lib REQUIRED)
find_path(LIVOX_LIDAR_SDK_INCLUDE_DIR
  NAMES "livox_lidar_api.h" "livox_lidar_def.h"
  REQUIRED)

link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

ament_auto_add_library(${PROJECT_NAME} SHARED
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

target_include_directories(${PROJECT_NAME} PUBLIC
  ${PCL_INCLUDE_DIRS}
  ${APR_INCLUDE_DIRS}
  ${LIVOX_LIDAR_SDK_INCLUDE_DIR}
  3rdparty
  src
)

target_link_libraries(${PROJECT_NAME}
  ${LIVOX_LIDAR_SDK_LIBRARY}
  ${PPT_LIBRARY}
  ${Boost_LIBRARY}
  ${PCL_LIBRARIES}
  ${APR_LIBRARIES}
)

rclcpp_components_register_node(${PROJECT_NAME}
  PLUGIN "livox_ros::DriverNode"
  EXECUTABLE ${PROJECT_NAME}_node
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_auto_package(INSTALL_TO_SHARE
  config
  launch_ROS2
)

endif()
