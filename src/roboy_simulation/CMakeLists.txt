cmake_minimum_required(VERSION 2.8.3)
project(roboy_simulation)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/devel/lib)
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -g")

SET(CMAKE_BUILD_TYPE Debug)

#==============================================================================
# Dependencies
#==============================================================================
find_package (Qt4)
find_package(Protobuf REQUIRED)
include (${QT_USE_FILE})
add_definitions(${QT_DEFINITIONS})

find_package(gazebo REQUIRED)
include_directories(${GAZEBO_INCLUDE_DIRS})
link_directories(${GAZEBO_LIBRARY_DIRS})
message(STATUS ${GAZEBO_INCLUDE_DIRS})

find_package(catkin REQUIRED COMPONENTS roscpp controller_manager gazebo_ros_control std_msgs geometry_msgs message_generation rviz)

add_message_files(
        DIRECTORY
        msgs
        FILES
	    Tendon.msg
        VisualizationControl.msg
)

generate_messages( DEPENDENCIES std_msgs geometry_msgs )

catkin_package(CATKIN_DEPENDS message_runtime message_generation)

include_directories(
	${catkin_INCLUDE_DIRS} 
	include/ 
	${CMAKE_CURRENT_SOURCE_DIR}/../common_utilities/include
	${CMAKE_CURRENT_SOURCE_DIR}/../flexrayusbinterface/include
)
#==============================================================================
# Application
#==============================================================================
add_library(muscle_plugin SHARED src/MusclePlugin.cpp)
target_link_libraries(muscle_plugin ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES} ${ARMADILLO_LIBRARIES} ${catkin_LIBRARIES})

add_library(tendon_visualizer SHARED src/TendonVisualizer.cpp)
add_dependencies(tendon_visualizer roboy_simulation_gencpp)
target_link_libraries(tendon_visualizer ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES})

add_library(roboySim SHARED src/roboySim.cpp)
add_dependencies(roboySim common_utilities_gencpp roboy_simulation_gencpp)
target_link_libraries(roboySim ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} ${Boost_LIBRARIES})

QT4_WRAP_CPP(headers_MOC include/GazeboRoboyOverlay.hpp)
add_library(GazeboRoboyOverlay SHARED ${headers_MOC} src/GazeboRoboyOverlay.cpp)
target_link_libraries(GazeboRoboyOverlay ${catkin_LIBRARIES} ${GAZEBO_LIBRARIES} ${QT_LIBRARIES} ${PROTOBUF_LIBRARIES})

qt4_wrap_cpp(MOC_FILES include/walking_plugin.hpp )
add_library(roboy_rviz src/walking_plugin.cpp ${MOC_FILES})
add_dependencies(roboy_rviz roboy_simulation_gencpp)
target_link_libraries(roboy_rviz ${QT_LIBRARIES} ${catkin_LIBRARIES})
#==============================================================================
# Installation
#==============================================================================
install(TARGETS
	muscle_plugin
	tendon_visualizer
	roboySim
	GazeboRoboyOverlay
	roboy_rviz    
	LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

install(FILES 
	package.xml
	muscle_plugin.xml
	walking_plugin.xml
 	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY 
	urdf/	
 	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/urdf
)

install(DIRECTORY 
	legs_with_muscles_simplified/	
 	DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/legs_with_muscles_simplified
)
