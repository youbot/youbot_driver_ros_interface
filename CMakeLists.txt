cmake_minimum_required(VERSION 2.8.3)
project(youbot_driver_ros_interface)


find_package(catkin REQUIRED COMPONENTS message_generation std_msgs roscpp actionlib tf control_msgs brics_actuator pr2_msgs)
find_package(Boost REQUIRED COMPONENTS thread date_time filesystem system)
find_package(youbot_driver REQUIRED)

catkin_package(
	CATKIN_DEPENDS pr2_msgs message_runtime brics_actuator
)

## preprocessor definitions
ADD_DEFINITIONS(-DYOUBOT_CONFIGURATIONS_DIR=${youbot_driver_CONFIG_DIR})

## Set Compiler flags
INCLUDE_DIRECTORIES(
	include
	${catkin_INCLUDE_DIRS}
	${Boost_INCLUDE_DIR}
	${youbot_driver_INCLUDE_DIRS}
)	


add_executable(youbot_driver_ros_interface 
    src/youbot_oodl.cpp 
    src/YouBotOODLWrapper.cpp
    src/YouBotConfiguration.cpp
    src/joint_state_observer_oodl.cpp
)

target_link_libraries(youbot_driver_ros_interface ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${youbot_driver_LIBRARIES})

add_dependencies(youbot_driver_ros_interface ${catkin_EXPORTED_TARGETS})

install(DIRECTORY launch/
	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
)


## run sudo command to enable direct network access
## disabled here by default because it is set by Debian postinst command

OPTION(USE_SETCAP "Set permissions to access ethernet interface without sudo" OFF)

SET(SUDO_COMMAND gksudo)
IF($ENV{USE_NORMAL_SUDO})
    SET(SUDO_COMMAND sudo)
ENDIF($ENV{USE_NORMAL_SUDO})

IF(USE_SETCAP)
add_custom_command(TARGET youbot_driver_ros_interface POST_BUILD
    COMMAND ${SUDO_COMMAND} setcap cap_net_raw+ep $<TARGET_FILE:youbot_driver_ros_interface>
)    
endif(USE_SETCAP)


#test programs
add_executable(youbot_arm_test 
    src/examples/youbot_arm_test.cpp 
)
target_link_libraries(youbot_arm_test ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${youbot_driver_LIBRARIES})

add_executable(youbot_2nd_arm_test 
	src/examples/youbot_2nd_arm_test.cpp 
)
target_link_libraries(youbot_2nd_arm_test ${catkin_LIBRARIES} ${Boost_LIBRARIES} ${youbot_driver_LIBRARIES})


install(TARGETS youbot_driver_ros_interface youbot_arm_test
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

