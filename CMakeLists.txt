# CMakeLists.txt
cmake_minimum_required(VERSION 3.10)
set (VCPKG_HOME "$ENV{HOME}/vcpkg") #IMPORTANT: change this to where your vcpkg exists.
set (CMAKE_TOOLCHAIN_FILE "${VCPKG_HOME}/scripts/buildsystems/vcpkg.cmake")
set (CMAKE_CXX_STANDARD 17)
set (CMAKE_BUILD_TYPE Debug)

project(HelloWorldMinimal)

find_package(gRPC CONFIG REQUIRED)
find_package(Protobuf CONFIG REQUIRED)
find_package(Threads REQUIRED)

add_subdirectory(protos)
add_subdirectory(src)
