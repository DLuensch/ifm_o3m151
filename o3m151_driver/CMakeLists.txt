cmake_minimum_required(VERSION 2.8.3)
project(o3m151_driver)

#Add custom (non compiling) targets so launch scripts and python files show up in QT Creator's project view.
file(GLOB EXTRA_FILES *.dox *.xml scripts/*.py launch/*.launch include/*/*.h action/*.action msg/*.msg)
add_custom_target(${PROJECT_NAME}_OTHER_FILES ALL WORKING_DIRECTORY ${PROJECT_SOURCE_DIR} SOURCES ${EXTRA_FILES})

set(${PROJECT_NAME}_CATKIN_DEPS 
    diagnostic_updater
    nodelet
    roscpp
    roslib
    pcl_ros
    sensor_msgs
    tf)

find_package(catkin REQUIRED COMPONENTS ${${PROJECT_NAME}_CATKIN_DEPS} pcl_conversions)

# This driver uses Boost threads
find_package(Boost REQUIRED COMPONENTS thread)

# libpcap provides no pkg-config or find_package module:
set(libpcap_LIBRARIES -lpcap)

include_directories(include ${Boost_INCLUDE_DIR} ${catkin_INCLUDE_DIRS})

# objects needed by other ROS packages that depend on this one
catkin_package(CATKIN_DEPENDS ${${PROJECT_NAME}_CATKIN_DEPS}
               INCLUDE_DIRS include
               LIBRARIES o3m151_input)

# compile the driver and input library
add_subdirectory(src/lib)
add_subdirectory(src/driver)

install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
install(FILES nodelet_o3m151.xml
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
install(DIRECTORY launch/
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch)
install(PROGRAMS src/vdump
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

if (CATKIN_ENABLE_TESTING)

  # these dependencies are only needed for unit testing
  find_package(roslaunch REQUIRED)
  find_package(rostest REQUIRED)
  
  # unit tests
  add_rostest(tests/pcap_node_hertz.test)
  add_rostest(tests/pcap_nodelet_hertz.test)
  add_rostest(tests/pcap_32e_node_hertz.test)
  add_rostest(tests/pcap_32e_nodelet_hertz.test)
  add_rostest(tests/pcap_vlp16_node_hertz.test)
  add_rostest(tests/pcap_vlp16_nodelet_hertz.test)
  
  # parse check all the launch/*.launch files
  roslaunch_add_file_check(launch)

endif (CATKIN_ENABLE_TESTING)
