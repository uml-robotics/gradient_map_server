cmake_minimum_required(VERSION 2.4.6 FATAL_ERROR)
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)
include($ENV{ROS_ROOT}/core/rosbuild/FindPkgConfig.cmake)
rosbuild_init()

## This plugin includes Qt widgets, so we must include Qt like so:
find_package(Qt4 COMPONENTS QtCore QtGui REQUIRED)
include(${QT_USE_FILE})

pkg_check_modules(OGRE OGRE) 
include_directories( ${OGRE_INCLUDE_DIRS} ) 
link_directories( ${OGRE_LIBRARY_DIRS} ) 

add_definitions(-DQT_NO_KEYWORDS)

qt4_wrap_cpp(MOC_FILES
  src/gradient_map_display.h
)

## Here we specify the list of source files, including the output of
## the previous command which is stored in ``${MOC_FILES}``.
set(SOURCE_FILES
  src/gradient_map_display.cpp
  ${MOC_FILES}
)

## Set the default path for built libraries to the "lib" directory
set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

rosbuild_add_library(gradient_map_display ${SOURCE_FILES})
set_target_properties(gradient_map_display PROPERTIES
  DEFINE_SYMBOL THIS_PACKAGE_NAME="${PROJECT_NAME}")
target_link_libraries(gradient_map_display ${QT_LIBRARIES} ${OGRE_LIBRARIES})

rosbuild_add_library(image_loader src/image_loader.cpp)
target_link_libraries(image_loader SDL SDL_image)

rosbuild_add_executable(bin/map_server src/main.cpp)
target_link_libraries(bin/map_server image_loader SDL SDL_image yaml-cpp)

rosbuild_add_gtest(test/utest test/utest.cpp test/test_constants.cpp)
target_link_libraries(test/utest image_loader SDL SDL_image)

rosbuild_add_executable(test/rtest test/rtest.cpp test/test_constants.cpp)
rosbuild_add_gtest_build_flags(test/rtest)

rosbuild_add_executable(bin/map_saver src/map_saver.cpp)

rosbuild_add_rostest(test/rtest.xml)
