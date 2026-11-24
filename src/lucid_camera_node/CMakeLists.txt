cmake_minimum_required(VERSION 3.8)
project(lucid_camera_node)

find_package(ament_cmake REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(OpenCV REQUIRED COMPONENTS core calib3d)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_msgs REQUIRED)
find_package(Threads REQUIRED)

set(ARENA_SDK_ROOT "$ENV{ARENA_SDK_ROOT}" CACHE PATH "LUCID Arena SDK for Linux x64")
if(NOT ARENA_SDK_ROOT)
  # Layout used by the original experiment workspace.
  get_filename_component(ARENA_SDK_ROOT
    "${CMAKE_CURRENT_SOURCE_DIR}/../../ArenaViewMP_v_1.0.0.10_Linux_x64/ArenaSDK_Linux_x64" ABSOLUTE)
endif()
set(arena_lib_dir "${ARENA_SDK_ROOT}/lib64")
set(genicam_lib_dir "${ARENA_SDK_ROOT}/GenICam/library/lib/Linux64_x64")
set(genicam_include_dir "${ARENA_SDK_ROOT}/GenICam/library/CPP/include")
if(NOT EXISTS "${ARENA_SDK_ROOT}/include/Arena/ArenaApi.h" OR
   NOT EXISTS "${arena_lib_dir}/libarena.so" OR NOT IS_DIRECTORY "${genicam_include_dir}")
  message(FATAL_ERROR "Cannot find Arena SDK at '${ARENA_SDK_ROOT}'. Set ARENA_SDK_ROOT to the Linux x64 SDK directory containing include/Arena and lib64.")
endif()

set(sdk_libraries "${arena_lib_dir}/libarena.so")
foreach(component GCBase GenApi Log)
  file(GLOB candidates "${genicam_lib_dir}/lib${component}_*_LUCID.so")
  list(LENGTH candidates count)
  if(NOT count EQUAL 1)
    message(FATAL_ERROR "Expected one LUCID ${component} library in ${genicam_lib_dir}, found ${count}. Check the SDK installation.")
  endif()
  list(APPEND sdk_libraries ${candidates})
endforeach()

add_executable(lucid_node src/lucid_node.cpp)
target_compile_features(lucid_node PRIVATE cxx_std_17)
target_include_directories(lucid_node PRIVATE
  "${ARENA_SDK_ROOT}/include" "${ARENA_SDK_ROOT}/include/Arena"
  "${genicam_include_dir}" ${OpenCV_INCLUDE_DIRS})
ament_target_dependencies(lucid_node ament_index_cpp cv_bridge rclcpp sensor_msgs std_msgs)
target_link_libraries(lucid_node ${OpenCV_LIBS} ${sdk_libraries} Threads::Threads)
set_target_properties(lucid_node PROPERTIES
  INSTALL_RPATH "${arena_lib_dir};${genicam_lib_dir}")

install(TARGETS lucid_node DESTINATION lib/${PROJECT_NAME})
install(FILES calibration/orientation.yml calibration/tritoncalibration.yml
  DESTINATION share/${PROJECT_NAME}/calibration)
ament_package()
