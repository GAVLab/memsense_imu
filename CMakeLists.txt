cmake_minimum_required(VERSION 2.8.3)
project(memsense_imu)

## Find catkin macros and libraries
## if COMPONENTS list like find_package(catkin REQUIRED COMPONENTS xyz)
## is used, also find other catkin packages
find_package(
    catkin REQUIRED
    COMPONENTS 
        message_generation
        driver_base
        dynamic_reconfigure
        nodelet
        roscpp
        sensor_msgs
)

## System dependencies are found with CMake's conventions
# find_package(Boost REQUIRED COMPONENTS system)


## Uncomment this if the package has a setup.py. This macro ensures
## modules and global scripts declared therein get installed
## See http://ros.org/doc/groovy/api/catkin/html/user_guide/setup_dot_py.html
# catkin_python_setup()

#######################################
## Declare ROS messages and services ##
#######################################

## Generate messages in the 'msg' folder
add_message_files(
    FILES
        ImuMAG.msg
)

## Generate services in the 'srv' folder
# add_service_files(
#   FILES
#   Service1.srv
#   Service2.srv
# )

## Generate added messages and services with any dependencies listed here
generate_messages(
    DEPENDENCIES
        sensor_msgs
)

#add dynamic reconfigure api
generate_dynamic_reconfigure_options(
    cfg/DynParams.cfg
    # cfg/cpp/memsense_imu/IMUDynParamsConfig.h
)


###################################
## catkin specific configuration ##
###################################
## The catkin_package macro generates cmake config files for your package
## Declare things to be passed to dependent projects
## LIBRARIES: libraries you create in this project that dependent projects also need
## CATKIN_DEPENDS: catkin_packages dependent projects also need
## DEPENDS: system dependencies of this project that dependent projects also need
catkin_package(
    INCLUDE_DIRS
        include
        include/IDC
        include/IMUDataUtils
        include/IMUSampler
        include/nodes
    LIBRARIES
        memsense_imu
    CATKIN_DEPENDS 
        driver_base
        dynamic_reconfigure
        nodelet
        roscpp
        sensor_msgs
    DEPENDS
        system_lib
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
## include dynamic_reconfigure header as well
include_directories(include
  ${catkin_INCLUDE_DIRS}
  cfg/cpp
)

## Set some sources
set(BASE_SOURCES
    src/nodes/imu_node_base.cpp
    src/nodes/imu_filter.cpp
    include/nodes/imu_filter.h
    include/nodes/imu_node_base.h
)

## Declare a cpp library
add_library(IMUDataUtils 
    src/IMUDataUtils/UnknownIMUSample.cpp
    src/IMUDataUtils/IMUCounterUtils.cpp
    src/IMUDataUtils/DeviceParams.cpp
    src/IMUDataUtils/StandardIMUHeader.cpp
    src/IMUDataUtils/IIMUPayload.cpp
    src/IMUDataUtils/SDTDefaultPre.cpp
    src/IMUDataUtils/IMUHiGPayload.cpp
    src/IMUDataUtils/SensorDataUtils.cpp
    src/IMUDataUtils/SerialDeviceParams.cpp
    src/IMUDataUtils/SDTDefault.cpp
    src/IMUDataUtils/CommonUtils.cpp
    src/IMUDataUtils/STDIMUSample.cpp
    src/IMUDataUtils/IMUTempPayload.cpp
    src/IMUDataUtils/SerialComm.cpp
    src/IMUDataUtils/IMUUnknownPayload.cpp
    src/IMUDataUtils/IMUMAGPayload.cpp
)
add_library(IMUSampler
    src/IMUSampler/IMUSampler.cpp
    include/IMUSampler/IMUSampler.h
)
add_library(imu_nodelet
    src/nodes/imu_nodelet.cpp
    ${BASE_SOURCES}
)


## Declare a cpp executable
add_executable(IDC
    src/IDC/AutoConfig.cpp
    include/IDC/AutoConfig.h
    src/IDC/GlobalVars.cpp
    include/IDC/GlobalVars.h
    src/IDC/GlobalVarsInternal.cpp
    include/IDC/GlobalVarsInternal.h
    include/IDC/IDCConfig.h
    src/IDC/IDCUtils.cpp
    include/IDC/IDCUtils.h
    include/IDC/IMenuHook.h
    src/IDC/IMUMsgsCommon.cpp
    include/IDC/IMUMsgsCommon.h
    include/IDC/IOHelpers.h
    include/IDC/IOHelpersImpl.h
    src/IDC/main.cpp
    src/IDC/MenuHelper.cpp
    include/IDC/MenuHelper.h
    src/IDC/MenuHelperInternal.cpp
    include/IDC/MenuHelperInternal.h
)
add_executable(imu_node
    src/nodes/imu_node.cpp
    ${BASE_SOURCES}
)

## Add cmake target dependencies of the executable/library
## as an example, message headers may need to be generated before nodes
# add_dependencies(memsense_imu_ex_node memsense_imu_ex_generate_messages_cpp)

# make sure configure headers are build before node using them
add_dependencies(imu_node
    memsense_imu_gencfg
) 



## Specify libraries to link a library or executable target against
# target_link_libraries(memsense_imu_ex_node
#   ${catkin_LIBRARIES}
# )
target_link_libraries(IDC 
    IMUDataUtils
    ${catkin_LIBRARIES}
)
target_link_libraries(IMUSampler
    IMUDataUtils
    ${catkin_LIBRARIES}
)
target_link_libraries(imu_node
    IMUSampler
    ${catkin_LIBRARIES}
)
target_link_libraries(imu_nodelet
    IMUSampler
    ${catkin_LIBRARIES}
)


#############
## Install ##
#############

# all install targets should use catkin DESTINATION variables
# See http://ros.org/doc/groovy/api/catkin/html/adv_user_guide/variables.html

## Mark executable scripts (Python etc.) for installation
## in contrast to setup.py, you can choose the destination
# install(PROGRAMS
#   scripts/my_python_script
#   DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark executables and/or libraries for installation
# install(TARGETS memsense_imu_ex memsense_imu_ex_node
#   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
#   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

## Mark cpp header files for installation
# install(DIRECTORY include/${PROJECT_NAME}/
#   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
#   FILES_MATCHING PATTERN "*.h"
#   PATTERN ".svn" EXCLUDE
# )

## Mark other files for installation (e.g. launch and bag files, etc.)
# install(FILES
#   # myfile1
#   # myfile2
#   DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
# )

#############
## Testing ##
#############

## Add gtest based cpp test target and link libraries
# catkin_add_gtest(${PROJECT_NAME}-test test/test_memsense_imu_ex.cpp)
# if(TARGET ${PROJECT_NAME}-test)
#   target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
# endif()

## Add folders to be run by python nosetests
# catkin_add_nosetests(test)


# add_subdirectory(src/IDC)
# add_subdirectory(src/IMUDataUtils)
# add_subdirectory(src/IMUSampler)
# include_directories(${PROJECT_SOURCE_DIR}/src)
# add_subdirectory(src/nodes)
# include_directories(${PROJECT_SOURCE_DIR}/src)













# ### Fuerte 


# include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)

# # Set the build type.  Options are:
# #  Coverage       : w/ debug symbols, w/o optimization, w/ code-coverage
# #  Debug          : w/ debug symbols, w/o optimization
# #  Release        : w/o debug symbols, w/ optimization
# #  RelWithDebInfo : w/ debug symbols, w/ optimization
# #  MinSizeRel     : w/o debug symbols, w/ optimization, stripped binaries
# #set(ROS_BUILD_TYPE RelWithDebInfo)

# rosbuild_init()

# #set the default path for built executables to the "bin" directory
# set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
# #set the default path for built libraries to the "lib" directory
# set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

# #uncomment if you have defined messages
# rosbuild_genmsg()
# #uncomment if you have defined services
# #rosbuild_gensrv()

# #add dynamic reconfigure api
# rosbuild_find_ros_package(dynamic_reconfigure)
# include(${dynamic_reconfigure_PACKAGE_PATH}/cmake/cfgbuild.cmake)
# gencfg()

# common commands for building c++ executables and libraries
# rosbuild_add_library(${PROJECT_NAME} src/example.cpp)
# target_link_libraries(${PROJECT_NAME} another_library)
# rosbuild_add_boost_directories()
# rosbuild_link_boost(${PROJECT_NAME} thread)
# rosbuild_add_executable(example examples/example.cpp)
# target_link_libraries(example ${PROJECT_NAME})

# add_subdirectory(src/IMUDataUtils)
# add_subdirectory(src/IMUSampler)
# add_subdirectory(src/IDC)
# add_subdirectory(src/nodes)
