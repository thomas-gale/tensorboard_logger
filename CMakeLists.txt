cmake_minimum_required(VERSION 3.17)

project(TensordboardLogger
    VERSION 0.1
    DESCRIPTION "C++ API to log data in tensorboard format"
    LANGUAGES CXX)

# Currently passing ${TENSOR_LOG_DIR} absolute path flag.
# Switch ${TENSOR_LOG_DIR} to ${PROJECT_SOURCE_DIR} ??

# Define proto directory and proto files.
set(PROTOBUF_DIRECTORY ${PROJECT_SOURCE_DIR}/proto)
file(GLOB PROTOBUF_DEFINITION_FILES ${PROTOBUF_DIRECTORY}/*.proto)

# Run protoc
execute_process(COMMAND protoc --proto_path=${PROTOBUF_DIRECTORY} --cpp_out=${PROTOBUF_DIRECTORY} ${PROTOBUF_DEFINITION_FILES})

# Collate generated headers and source file
file(GLOB PROTOBUF_GEN_HEADERS ${PROTOBUF_DIRECTORY}/*.h)
file(GLOB PROTOBUF_GEN_SRC ${PROTOBUF_DIRECTORY}/*.cc)

# Move files
execute_process(COMMAND mv ${PROTOBUF_GEN_HEADERS} ${PROJECT_SOURCE_DIR}/include)
execute_process(COMMAND mv ${PROTOBUF_GEN_SRC} ${PROJECT_SOURCE_DIR}/src)

# Define library 
file(GLOB HEADER_LIST ${PROJECT_SOURCE_DIR}/include/*.h)
file(GLOB SRC_LIST ${PROJECT_SOURCE_DIR}/src/*.cc)

add_library(tensorboard_logger
    ${HEADER_LIST}
    ${SRC_LIST})

find_library(protobuf REQUIRED)

target_include_directories(tensorboard_logger
    PUBLIC 
    ${PROJECT_SOURCE_DIR}/include
    ${PROTOBUF_include})

target_link_libraries(tensorboard_logger 
    PUBLIC
    protobuf)

# All users of this library will need at least C++11
#target_compile_features(tensorboard_logger PUBLIC cxx_std_11)
#target_compile_features(tensorboard_logger PRIVATE -Wall)

# Define Test Library
add_executable(tensorboard_logger_tests
    ${HEADER_LIST}
    tests/test_tensorboard_logger.cc)

target_link_libraries(tensorboard_logger_tests 
    PRIVATE
    tensorboard_logger)
