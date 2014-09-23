cmake_minimum_required(VERSION 2.8.3)
project(vicon_bridge)

find_package(catkin REQUIRED COMPONENTS
  roscpp message_generation geometry_msgs tf diagnostic_updater dynamic_reconfigure
)
find_package(Boost REQUIRED COMPONENTS signals)

add_service_files(
  FILES
  viconCalibrateSegment.srv
  viconGrabPose.srv
)

add_message_files(
  FILES
  Marker.msg
  Markers.msg
  TfDistortInfo.msg
)

generate_messages(
  DEPENDENCIES
  geometry_msgs
)

generate_dynamic_reconfigure_options(cfg/tf_distort.cfg)

catkin_package(
  CATKIN_DEPENDS roscpp message_runtime geometry_msgs tf diagnostic_updater
)

if(CMAKE_SIZEOF_VOID_P MATCHES "8")
    set(VICON_SDK             vicon_sdk/vicon_datastream_SDK_1.2.59611_x64)
    set(VICON_SDK_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/${VICON_SDK})
    set(VICON_SDK_LINK_DIR    ${PROJECT_SOURCE_DIR}/${VICON_SDK})
elseif(CMAKE_SIZEOF_VOID_P MATCHES "4")
    set(VICON_SDK             vicon_sdk/vicon_datastream_SDK_1.2.59611_x86)
    set(VICON_SDK_INCLUDE_DIR ${PROJECT_SOURCE_DIR}/${VICON_SDK})
    set(VICON_SDK_LINK_DIR    ${PROJECT_SOURCE_DIR}/${VICON_SDK})
else(CMAKE_SIZEOF_VOID_P MATCHES "8")
    message(FATAL_ERROR "unable to detect platform")
endif (CMAKE_SIZEOF_VOID_P MATCHES "8")

include_directories(
  ${catkin_INCLUDE_DIRS}
  ${VICON_SDK_INCLUDE_DIR}
)

link_directories(${VICON_SDK_LINK_DIR})

add_library(msvc_bridge src/msvc_bridge.cpp)

add_executable(vicon_bridge src/vicon_bridge.cpp)
add_dependencies(vicon_bridge vicon_bridge_gencpp ${${PROJECT_NAME}_EXPORTED_TARGETS})
target_link_libraries(vicon_bridge
  ${catkin_LIBRARIES} msvc_bridge ViconDataStreamSDK_CPP
)

add_executable(calibrate src/calibrate_segment.cpp)
add_dependencies(calibrate vicon_bridge_gencpp ${${PROJECT_NAME}_EXPORTED_TARGETS})
target_link_libraries(calibrate
  ${catkin_LIBRARIES}
)

add_executable(tf_distort src/tf_distort.cpp)
add_dependencies(tf_distort vicon_bridge_gencpp ${${PROJECT_NAME}_EXPORTED_TARGETS})
target_link_libraries(tf_distort
  ${catkin_LIBRARIES}
)

add_executable(testclient src/ViconDataStreamSDK_CPPTest.cpp)
target_link_libraries(testclient
  ${catkin_LIBRARIES} ViconDataStreamSDK_CPP
)
