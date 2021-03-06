cmake_minimum_required( VERSION 2.8 )
project( RobotSystem-Lite )

set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR} )

set( CMAKE_C_STANDARD 99 )
set( CMAKE_C_STANDARD_REQUIRED ON )

set( MODULES_PATH plugins )

set( SOURCES_DIR ${CMAKE_SOURCE_DIR}/src/ )
set( PLUGIN_SOURCES_DIR ${SOURCES_DIR}/${MODULES_PATH} )

set( LIBRARY_DIR ${CMAKE_SOURCE_DIR}/libs CACHE PATH "Linked libraries directory" FORCE )
set( MODULES_DIR ${CMAKE_SOURCE_DIR}/${MODULES_PATH} CACHE PATH "Dynamic modules output directory" FORCE )

set( SIGNAL_IO_PATH signal_io )
set( ROBOT_CONTROL_PATH robot_control )

include( ${SOURCES_DIR}/debug/CMakeLists.txt )
include( ${SOURCES_DIR}/data_io/CMakeLists.txt )
include( ${SOURCES_DIR}/kalman/CMakeLists.txt )
include( ${SOURCES_DIR}/signal_processing/CMakeLists.txt )
include( ${SOURCES_DIR}/threads/CMakeLists.txt )
include( ${SOURCES_DIR}/timing/CMakeLists.txt )
include( ${SOURCES_DIR}/ipc/CMakeLists.txt )

include_directories( ${SOURCES_DIR} )
link_directories( ${LIBRARY_DIR} )


# (REAL-TIME) CONTROL APPLICATION
add_executable( RobotControl ${SOURCES_DIR}/main.c ${SOURCES_DIR}/system.c ${SOURCES_DIR}/robot.c ${SOURCES_DIR}/actuator.c ${SOURCES_DIR}/sensor.c ${SOURCES_DIR}/motor.c )
target_compile_definitions( RobotControl PUBLIC -DDEBUG -DZMQ_BUILD_DRAFT_API )
target_link_libraries( RobotControl DataLogging DataIOJSON KalmanFilters SignalProcessing AsyncIPC MultiThreading Timing ${CMAKE_DL_LIBS} )


# EXAMPLE PLUGINS/MODULES

add_library( DummyIO MODULE ${PLUGIN_SOURCES_DIR}/${SIGNAL_IO_PATH}/dummy.c )
set_target_properties( DummyIO PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${MODULES_DIR}/${SIGNAL_IO_PATH} )
set_target_properties( DummyIO PROPERTIES PREFIX "" )
target_include_directories( DummyIO PUBLIC ${PLUGIN_SOURCES_DIR}/${SIGNAL_IO_PATH}/ )
 
add_library( SimpleJoint MODULE ${PLUGIN_SOURCES_DIR}/${ROBOT_CONTROL_PATH}/simple_joint.c )
set_target_properties( SimpleJoint PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${MODULES_DIR}/${ROBOT_CONTROL_PATH} )
set_target_properties( SimpleJoint PROPERTIES PREFIX "" )
target_include_directories( SimpleJoint PUBLIC ${PLUGIN_SOURCES_DIR}/${ROBOT_CONTROL_PATH}/ )
 
# add_library( AnkleBot MODULE ${SOURCES_DIR}/robot_control/anklebot.c )
# set_target_properties( AnkleBot PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${ROBOT_CONTROL_MODULES_DIR} )
# set_target_properties( AnkleBot PROPERTIES PREFIX "" )
# target_include_directories( AnkleBot PUBLIC ${SOURCES_DIR}/robot_control/ )
# 
# add_library( EMGJoint MODULE ${SOURCES_DIR}/robot_control/emg/emg_control.c ${SOURCES_DIR}/robot_control/emg/emg_muscular_model.c )
# set_target_properties( EMGJoint PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${ROBOT_CONTROL_MODULES_DIR} )
# set_target_properties( EMGJoint PROPERTIES PREFIX "" )
# target_include_directories( EMGJoint PUBLIC ${SOURCES_DIR}/robot_control/ ${SOURCES_DIR}/robot_control/emg/ )

# install( TARGETS RobRehabGui RobRehabServer RUNTIME DESTINATION . )
