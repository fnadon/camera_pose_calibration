cmake_minimum_required(VERSION 3.8)
project(camera_pose_calibration)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

cmake_policy(SET CMP0074 OLD)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(cv_bridge REQUIRED)
# find_package(eigen_conversions REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(image_transport REQUIRED)
#find_package(message_generation REQUIRED)
#find_package(nodelet REQUIRED)
find_package(pcl_ros REQUIRED)
#find_package(roscpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_sensor_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
# find_package(catkin REQUIRED COMPONENTS
# 	cv_bridge
# 	eigen_conversions
# 	geometry_msgs
# 	image_transport
# 	message_generation
# 	nodelet
# 	pcl_ros
# 	roscpp
# 	sensor_msgs
# 	tf
# 	tf_conversions
# 	visualization_msgs
# )

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(PCL REQUIRED)
find_package(Boost REQUIRED)

add_definitions(${PCL_DEFINITIONS})

# messages
find_package(rosidl_default_generators REQUIRED)
rosidl_generate_interfaces(${PROJECT_NAME}
  # add interfaces to generate here
  #"msg/message.msg" or "srv/server.srv"
  "srv/CalibrateCall.srv"
  "srv/CalibrateFile.srv"
  "srv/CalibrateTopic.srv"
  "msg/PatternParameters.msg"
)

# add_message_files(DIRECTORY msg)
# add_service_files(DIRECTORY srv)
# generate_messages(DEPENDENCIES
# 	geometry_msgs
# 	sensor_msgs
# )

#ament_package()

if(TARGET tf2_geometry_msgs::tf2_geometry_msgs)
  get_target_property(_include_dirs tf2_geometry_msgs::tf2_geometry_msgs INTERFACE_INCLUDE_DIRECTORIES)
else()
  set(_include_dirs ${tf2_geometry_msgs_INCLUDE_DIRS})
endif()

find_file(TF2_CPP_HEADERS
  NAMES tf2_geometry_msgs.hpp
  PATHS ${_include_dirs}
  NO_CACHE
  PATH_SUFFIXES tf2_geometry_msgs
)

include_directories(
	include/${PROJECT_NAME}
SYSTEM
	# ${catkin_INCLUDE_DIRS}
	${Boost_INCLUDE_DIRS}
	${EIGEN3_INCLUDE_DIRS}
	${OpenCV_INCLUDE_DIRS}
	${PCL_INCLUDE_DIRS}
)

add_executable(${PROJECT_NAME}_node src/main.cpp)
ament_target_dependencies(${PROJECT_NAME}_node
  rclcpp
  cv_bridge
#   eigen_conversions
  geometry_msgs
  image_transport
#   message_generation
  pcl_ros
  sensor_msgs
  tf2_ros
  tf2_sensor_msgs
  visualization_msgs
)
target_include_directories(${PROJECT_NAME}_node PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
target_compile_features(${PROJECT_NAME}_node PUBLIC c_std_99 cxx_std_17)  # Require C99 and C++17

install(TARGETS ${PROJECT_NAME}_node
  DESTINATION lib/${PROJECT_NAME})

rosidl_target_interfaces(${PROJECT_NAME}_node
${PROJECT_NAME} "rosidl_typesupport_cpp")
# add_library(${PROJECT_NAME} src/${PROJECT_NAME}.cpp)
# target_link_libraries(${PROJECT_NAME}
# 	# ${catkin_LIBRARIES}
# 	${EIGEN3_LIBRARIES}
# 	${OpenCV_LIBRARIES}
# 	${PCL_LIBRARIES}
# )

# add_library(${PROJECT_NAME}_nodelet SHARED
# 	src/node.cpp
# 	src/nodelet.cpp
# )
# target_link_libraries(${PROJECT_NAME}_nodelet
# 	${PROJECT_NAME}
# )
# add_dependencies(${PROJECT_NAME}_nodelet ${PROJECT_NAME}_gencpp)

# add_executable(${PROJECT_NAME}_node src/main.cpp)
# target_link_libraries(${PROJECT_NAME}_node ${PROJECT_NAME}_nodelet)

# if(CATKIN_ENABLE_TESTING)
# 	catkin_add_gtest(${PROJECT_NAME}_test test/${PROJECT_NAME}.cpp)
# 	target_link_libraries(${PROJECT_NAME}_test ${catkin_LIBRARIES} ${PROJECT_NAME})
# endif()

# install(TARGETS ${PROJECT_NAME} ${PROJECT_NAME}_node ${PROJECT_NAME}_nodelet
# 	ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
# 	LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
# 	RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
# )

# install(DIRECTORY launch DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
# install(FILES nodelet_plugins.xml DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

ament_package()