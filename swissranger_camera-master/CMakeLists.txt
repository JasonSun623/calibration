cmake_minimum_required(VERSION 2.8.3)
project(swissranger_camera)
set(CMAKE_BUILD_TYPE RelWithDebInfo)
find_package(catkin REQUIRED COMPONENTS
  cmake_modules roscpp tf camera_info_manager image_transport
  dynamic_reconfigure driver_base sensor_msgs cv_bridge
  nodelet)

find_package(OpenCV REQUIRED)
find_package(Boost REQUIRED COMPONENTS signals thread)

include_directories(${OpenCV_INCLUDE_DIRS})
link_directories(${OpenCV_LIB_DIR})
add_definitions(${OpenCV_DEFINITIONS})

find_package(Eigen REQUIRED)
include_directories(${Eigen_INCLUDE_DIRS} include ${catkin_INCLUDE_DIRS})

find_package(PCL 1.7 REQUIRED)
include_directories(BEFORE ${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})
MESSAGE("PCL_INCLUDE_DIRS\n${PCL_INCLUDE_DIRS}\n")
MESSAGE("PCL_LIBRARY_DIRS\n${PCL_LIBRARY_DIRS}\n")
MESSAGE("PCL_DEFINITIONS\n${PCL_DEFINITIONS}\n")
MESSAGE("PCL_COMMON_LIBRARIES\n${PCL_COMMON_LIBRARIES}\n")
if (NOT PCL_FOUND)
    MESSAGE(FATAL_ERROR "PCL not found.\n")
endif (NOT PCL_FOUND)

generate_dynamic_reconfigure_options(cfg/SwissRanger.cfg)

include_directories(include ${catkin_INCLUDE_DIRS} cfg/cpp)

catkin_package(
   INCLUDE_DIRS
   LIBRARIES ${PROJECT_NAME}
   CATKIN_DEPENDS roscpp tf camera_info_manager image_transport dynamic_reconfigure driver_base
)

add_executable(${PROJECT_NAME} src/sr.cpp src/dev_sr.cpp)
add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}_gencfg)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES} mesasr)

add_executable(image_publisher_sr apps/main_image_publisher_sr.cpp)
add_dependencies(image_publisher_sr ${PROJECT_NAME}_gencfg)
target_link_libraries(image_publisher_sr ${catkin_LIBRARIES})

add_executable(intrinsics_params_extractor src/intrinsics_params_extractor.cpp)
target_link_libraries(intrinsics_params_extractor ${OpenCV_LIBRARIES} mesasr)

add_executable(depth_view src/depth_view.cpp)
target_link_libraries(depth_view ${catkin_LIBRARIES})

add_library(depth_view_nodelet src/nodelets/image_nodelet.cpp src/nodelets/window_thread.cpp)

target_link_libraries(depth_view_nodelet ${catkin_LIBRARIES}
                                 ${OpenCV_LIBRARIES}
                                 ${Boost_LIBRARIES}
)


install(TARGETS ${PROJECT_NAME}
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS image_publisher_sr
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS intrinsics_params_extractor
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS depth_view
        DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS depth_view_nodelet
        DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

install(FILES nodelet_plugins.xml
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)
