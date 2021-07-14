cmake_minimum_required(VERSION 3.0.2)
project(scoped_roslaunch)

find_package(catkin REQUIRED)

catkin_package()

catkin_install_python(PROGRAMS
  scripts/roslaunch
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
