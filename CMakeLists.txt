cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(rangenet_lib)

# handle debug vs release mode
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_FLAGS "-Wall")
set(CMAKE_CXX_FLAGS_DEBUG "-g -O0")
set(CMAKE_CXX_FLAGS_RELEASE "-g -O3")

## Use C++17
set(CMAKE_CXX_STANDARD 17)

# check flags
message("Build type: ${CMAKE_BUILD_TYPE}")

## Find catkin macros and libraries
find_package(catkin REQUIRED)

# OpenCV (required)
find_package(OpenCV REQUIRED)
if(OpenCV_FOUND)
  message("OpenCV Libs: ${OpenCV_LIBS}")
else()
  message(FATAL_ERROR "OpenCV NOT FOUND\n")
endif()

# libyaml-cpp (required)
find_package(yaml-cpp REQUIRED)
if (yaml-cpp_FOUND)
  message("YAML Libs: ${YAML_CPP_LIBRARIES}")
else()
  message(FATAL_ERROR "yaml-cpp NOT FOUND\n")
endif()

# Boost (required)
find_package(Boost COMPONENTS program_options filesystem REQUIRED)
if(Boost_FOUND)
  message("Boost Libs: ${Boost_LIBRARIES}")
else()
  message(FATAL_ERROR "Boost NOT FOUND\n")
endif()

# CUDA (required)
set(CUDA_TOOLKIT_ROOT_DIR /usr/local/cuda-11.7)
find_package(CUDA REQUIRED)
if(CUDA_FOUND)
  message("CUDA SUCCESS!\n")
  set(CUDA_INCLUDE_DIRS /usr/local/cuda-11.7/include)
  set(CUDA_CUDART_LIBRARY /usr/local/cuda-11.7/lib64/libcudart.so)  # Dynamic linking
  include_directories(${CUDA_INCLUDE_DIRS})
else()
  message(FATAL_ERROR "CUDA NOT FOUND\n")
endif()

# TensorRT (required)
set(TENSORRT_INCLUDE_DIR /home/lognav/TensorRT-8.4.3.1/targets/x86_64-linux-gnu/include)
set(TENSORRT_LIBRARY_DIR /home/lognav/TensorRT-8.4.3.1/targets/x86_64-linux-gnu/lib)
message("CUDA Include Path: ${CUDA_INCLUDE_DIRS}")
message("CUDA Library: ${CUDA_CUDART_LIBRARY}")
message("TensorRT Include Path: ${TENSORRT_INCLUDE_DIR}")
message("TensorRT Library Path: ${TENSORRT_LIBRARY_DIR}")

find_library(TENSORRT_LIB nvinfer PATHS ${TENSORRT_LIBRARY_DIR})
find_library(NVONNX_PARSER nvonnxparser PATHS ${TENSORRT_LIBRARY_DIR})

if(TENSORRT_LIB AND NVONNX_PARSER)
  message("TensorRT and ONNX parser found!")
else()
  message(FATAL_ERROR "TensorRT libraries NOT FOUND\n")
endif()


###################################
## catkin specific configuration ##
###################################
catkin_package(
  INCLUDE_DIRS include
  LIBRARIES rangenet_external rangenet_lib
  CATKIN_DEPENDS
  DEPENDS YAML_CPP
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${YAML_CPP_INCLUDE_DIR}
  ${OpenCV_INCLUDE_DIRS}
  ${CUDA_INCLUDE_DIRS}
  ${TENSORRT_INCLUDE_DIR}
)

# Library for external dependencies
add_library(rangenet_external src/external.cpp)

# rangenet_lib
set(rangenet_lib_SOURCES src/net.cpp src/selector.cpp)
if(TENSORRT_LIB)
  list(APPEND rangenet_lib_SOURCES src/netTensorRT.cpp)
endif()

add_library(rangenet_lib ${rangenet_lib_SOURCES})
target_link_libraries(rangenet_lib
  rangenet_external
  ${catkin_LIBRARIES}
  ${YAML_CPP_LIBRARIES}
  ${OpenCV_LIBS}
  ${CUDA_CUDART_LIBRARY}    # Link to libcudart_static.a
  ${TENSORRT_LIB}
  ${NVONNX_PARSER}
)

# Build example if enabled
option(EXAMPLE "Building example" ON)
if(EXAMPLE)
  add_executable(infer example/infer.cpp)
  target_link_libraries(infer
    rangenet_lib
    ${catkin_LIBRARIES}
    ${OpenCV_LIBS}
    ${Boost_LIBRARIES}
    ${YAML_CPP_LIBRARIES}
    ${CUDA_CUDART_LIBRARY}    # Link to libcudart_static.a
    ${TENSORRT_LIB}
    ${NVONNX_PARSER}
  )
endif()
