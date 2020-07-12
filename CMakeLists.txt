cmake_minimum_required(VERSION 3.17)

project(TensordboardLogger
    VERSION 0.1
    DESCRIPTION "C++ API to log data in tensorboard format"
    LANGUAGES CXX)

set (CMAKE_CXX_STANDARD 17)

# Define proto directory and proto files.
set(PROTOBUF_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/proto)
file(GLOB PROTOBUF_DEFINITION_FILES ${PROTOBUF_DIRECTORY}/*.proto)

# Run protoc
execute_process(COMMAND protoc --proto_path=${PROTOBUF_DIRECTORY} --cpp_out=${PROTOBUF_DIRECTORY} ${PROTOBUF_DEFINITION_FILES})

# Collate generated headers and source file
file(GLOB PROTOBUF_GEN_HEADERS ${PROTOBUF_DIRECTORY}/*.h)
file(GLOB PROTOBUF_GEN_SRC ${PROTOBUF_DIRECTORY}/*.cc)

# Move files
execute_process(COMMAND mv ${PROTOBUF_GEN_HEADERS} ${CMAKE_CURRENT_SOURCE_DIR}/include)
execute_process(COMMAND mv ${PROTOBUF_GEN_SRC} ${CMAKE_CURRENT_SOURCE_DIR}/src)

# Define library 
file(GLOB HEADER_LIST ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
file(GLOB SRC_LIST ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc)

# Locate protobuffer library (Built and located in /usr/lib/... during Docker environemnt setup)
include(FindProtobuf)
find_package(Protobuf REQUIRED)

add_library(tensorboard_logger
    ${HEADER_LIST}
    ${SRC_LIST})

target_include_directories(tensorboard_logger
    PUBLIC 
    ${PROJECT_SOURCE_DIR}/include)

target_link_libraries(tensorboard_logger 
    PUBLIC
    ${PROTOBUF_LIBRARIES}
    stdc++fs)

# All users of this library will need at least C++11
#target_compile_features(tensorboard_logger PUBLIC cxx_std_11)
#target_compile_features(tensorboard_logger PRIVATE -Wall)

# Define Test Library
add_executable(tensorboard_logger_tests
    ${HEADER_LIST}
    tests/test_tensorboard_logger.cc)

target_include_directories(tensorboard_logger_tests
    PRIVATE 
    ${PROJECT_SOURCE_DIR}/include)

target_link_libraries(tensorboard_logger_tests 
    PRIVATE
    tensorboard_logger)

# Copy test assets

