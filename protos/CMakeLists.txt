#Adapted from: https://github.com/matrix-io/protocol-buffers/blob/master/CMakeLists.txt
cmake_minimum_required(VERSION 3.10)

project(HelloWorldMinimal)

find_program(PROTOC NAMES protoc)
if (NOT EXISTS ${PROTOC})
  message(FATAL_ERROR "The protoc program was not found")
endif()
message(STATUS "Found protoc program: " ${PROTOC})

# protoc grpc plugin
find_program(GRPC_PLUGIN NAMES "grpc_cpp_plugin")
if (NOT EXISTS ${GRPC_PLUGIN})
  message(WARNING "The grpc_cpp_plugin plugin was not found, \
                   the gRPC classes are not being generated")
else(EXISTS ${GRPC_PLUGIN})
  message(STATUS "Found grpc_cpp_plugin : " ${GRPC_PLUGIN})
endif()

# .proto file search directory
if (NOT DEFINED PROTO_SEARCH_PATH)
  set(PROTO_SEARCH_PATH ${CMAKE_CURRENT_SOURCE_DIR})
endif()

# protoc flags
set(PROTO_CFLAGS)
list(APPEND PROTOC_FLAGS "--proto_path=${PROTO_SEARCH_PATH}")
list(APPEND PROTOC_FLAGS "--cpp_out=${CMAKE_CURRENT_BINARY_DIR}") 

# Build grpc if the plugin is found
if (EXISTS ${GRPC_PLUGIN})
  list(APPEND PROTOC_FLAGS "--plugin=protoc-gen-grpc=${GRPC_PLUGIN}")
  list(APPEND PROTOC_FLAGS "--grpc_out=${CMAKE_CURRENT_BINARY_DIR}") 
endif()

# .proto relative paths
file(GLOB_RECURSE proto_files RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} *.proto)

# Absolute paths of .proto files to be passed to protoc program
set(ABSProtoFiles)
# Absolute paths of generated .h and .cc 
set(ProtoHeaders)
set(ProtoSources)

include_directories(${ProtoHeaders})

foreach(FNAME ${proto_files})
  # relative directory for .proto file
  get_filename_component(PROTO_PATH ${FNAME} DIRECTORY)
  # .proto name
  get_filename_component(PROTO_NAME ${FNAME} NAME_WE)
  # absolute directory and name of generated .h and .cc file
  set(GENERATED_PROTO_PATH "${CMAKE_CURRENT_BINARY_DIR}/${PROTO_PATH}/${PROTO_NAME}")

  list(APPEND ABSProtoFiles "${PROTO_SEARCH_PATH}/${FNAME}")
  list(APPEND ProtoHeaders "${GENERATED_PROTO_PATH}.pb.h")
  list(APPEND ProtoSources "${GENERATED_PROTO_PATH}.pb.cc")
  install (FILES ${GENERATED_PROTO_PATH}.pb.h DESTINATION include/${PROTO_PATH})

  # include grpc .h and .cc files if generated
  if (EXISTS ${GRPC_PLUGIN})
    list(APPEND ProtoHeaders "${GENERATED_PROTO_PATH}.grpc.pb.h")
    list(APPEND ProtoSources "${GENERATED_PROTO_PATH}.grpc.pb.cc")
    install (FILES "${GENERATED_PROTO_PATH}.grpc.pb.h" DESTINATION "include/${PROTO_PATH}")
  endif()
endforeach()

# Generate protos
add_custom_command(
  COMMAND ${PROTOC} ${PROTOC_FLAGS} ${ABSProtoFiles}
  OUTPUT ${ProtoSources} ${ProtoHeaders}
  COMMENT "Generating proto messages ..."
  DEPENDS ${ABSProtoFiles}
)

add_library(p3protolib ${ProtoHeaders} ${ProtoSources})
target_link_libraries(p3protolib PUBLIC protobuf::libprotobuf)
target_include_directories(p3protolib PUBLIC ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_BINARY_DIR}/protos)

