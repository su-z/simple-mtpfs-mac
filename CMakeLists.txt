cmake_minimum_required(VERSION 3.17)
project(mtpfs)
set(MTPFS_NAME simple-mtpfs)

set(CMAKE_CXX_STANDARD 17)
set(FUSE_OPTIONS -D_FILE_OFFSET_BITS=64 -DFUSE_USE_VERSION=28 -DDEBUG_V8)

set(LIBUSB_LIBRARIES)
set(LIBUSB_INCLUDE_DIRS)

set(FUSE_INCLUDE_DIRS)
set(LIBMTP_INCLUDE_DIRS)


add_compile_options(${FUSE_OPTIONS} -framework IOKit)

file (GLOB my_sources "./src/*.cpp")
add_executable(${MTPFS_NAME} ${my_sources})
target_include_directories(${MTPFS_NAME} PUBLIC ${FUSE_INCLUDE_DIRS} ${LIBUSB_INCLUDE_DIRS} ${LIBMTP_INCLUDE_DIRS})

find_library(LIBMTP mtp REQUIRED)

find_library(LIBFUSE fuse REQUIRED)

target_link_libraries(${MTPFS_NAME} ${LIBUSB_LIBRARIES} ${LIBMTP} ${LIBFUSE})
target_include_directories(${MTPFS_NAME} PUBLIC ./)
target_include_directories(${MTPFS_NAME} PUBLIC ${FUSE_INCLUDE_DIRS})
