cmake_minimum_required(VERSION 3.10)
project(orbslam3)

########################
# 1.  Global settings  #
########################
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_compile_options(-Wall -Wextra)


# Clean any conflicting OpenCV settings
unset(OpenCV_FOUND CACHE)
unset(OpenCV_INCLUDE_DIRS CACHE)
unset(OpenCV_LIBS CACHE)

# Use default system OpenCV
set(OpenCV_DIR "/usr/lib/x86_64-linux-gnu/cmake/opencv4")
find_package(OpenCV 4.2 REQUIRED)

########################
# 2.  ROS2 + deps      #
########################
find_package(ament_cmake REQUIRED)
find_package(rclcpp        REQUIRED)
find_package(sensor_msgs   REQUIRED)
find_package(message_filters REQUIRED)
find_package(cv_bridge     REQUIRED)
find_package(OpenCV 4.2 REQUIRED)
find_package(Pangolin      REQUIRED)

# -------- ORB-SLAM 3 (already installed under /usr/local) -------
find_package(ORB_SLAM3 REQUIRED CONFIG)

# -------- ONNXRuntime helper target ----------------------------
include(${CMAKE_CURRENT_SOURCE_DIR}/onnx_import.cmake)   # creates onnxruntime::onnxruntime

##############################################################
# 3.  Include directories that the wrapper code needs
##############################################################
include_directories(
  include
  /usr/local/include                     # << contains KeyFrame.h,
  /usr/local/include/CameraModels
  ${ORB_SLAM3_ROOT_DIR}/include
  ${ORB_SLAM3_ROOT_DIR}/include/CameraModels
  ${ORB_SLAM3_ROOT_DIR}/Thirdparty/Sophus      # headers were exported here
  /usr/local/include/Thirdparty/Sophus 
)


##############################################################
# 4.  Common libraries we’ll link against
##############################################################
set(COMMON_LIBS
  ORB_SLAM3::ORB_SLAM3          # pulls in DBoW3 / g2o
  onnxruntime::onnxruntime

  ${Pangolin_LIBRARIES}
)

##############################################################
# 5.  Helper macro to make the four demo nodes
##############################################################
macro(make_slam_node target)
  add_executable(${ARGV})
  target_link_libraries(${target} ${COMMON_LIBS})
  ament_target_dependencies(${target}
      rclcpp sensor_msgs message_filters cv_bridge)
  install(TARGETS ${target} DESTINATION lib/${PROJECT_NAME})
endmacro()

make_slam_node(mono            src/monocular/mono               src/monocular/monocular-slam-node.cpp)
make_slam_node(stereo_inertial src/stereo-inertial/stereo-inertial.cpp src/stereo-inertial/stereo-inertial-node.cpp)
make_slam_node(mono_inertial   src/mono-inertial/mono-inertial.cpp   src/mono-inertial/monocular-inertial-slam-node.cpp)
ament_package()

