#
# (c) 2019 Copyright, Real-Time Innovations, Inc.  All rights reserved.
#
#  RTI grants Licensee a license to use, modify, compile, and create derivative
#  works of the Software.  Licensee has the right to distribute object form
#  only for use with RTI products.  The Software is provided "as is", with no
#  warranty of any type, including any warranty for fitness for any purpose.
#  RTI is under no obligation to maintain or support the Software.  RTI shall
#  not be liable for any incidental or consequential damages arising out of the
#  use or inability to use the software.
#
cmake_minimum_required(VERSION 3.11)
project (MongoDBAdapter)

# Find RTI Connext dependencies
set(CMAKE_MODULE_PATH
    ${CMAKE_MODULE_PATH}
    "${CMAKE_CURRENT_SOURCE_DIR}/../../../../resources/cmake"
)

find_package(
    RTIConnextDDS "6.1.1"
    REQUIRED
    COMPONENTS
        core
        routing_service
)

# Mongo DB C++ Driver
set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} ${MONGODB_DIR})
find_package(mongocxx REQUIRED)

add_library(
    mongodbadapter
        "${CMAKE_CURRENT_SOURCE_DIR}/SampleConverter.cxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/MongoConfig.cxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/MongoAdapter.cxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/MongoConnection.cxx"
        "${CMAKE_CURRENT_SOURCE_DIR}/MongoStreamWriter.cxx" MongoStreamReader.cpp MongoStreamReader.hpp)

# Set target properties for lang requirement output library name
set_target_properties(mongodbadapter
    PROPERTIES
        CXX_STANDARD 11
        CXX_STANDARD_REQUIRED ON
        OUTPUT_NAME_DEBUG mongodbadapterd
        LIBRARY_OUTPUT_DIRECTORY "${output_dir}"
        LIBRARY_OUTPUT_DIRECTORY_RELEASE "${output_dir}"
        LIBRARY_OUTPUT_DIRECTORY_DEBUG "${output_dir}"
        ARCHIVE_OUTPUT_DIRECTORY "${output_dir}"
        ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${output_dir}"
        ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${output_dir}"
        RUNTIME_OUTPUT_DIRECTORY "${output_dir}"
        RUNTIME_OUTPUT_DIRECTORY_RELEASE "${output_dir}"
        RUNTIME_OUTPUT_DIRECTORY_DEBUG "${output_dir}"
)

target_link_libraries(
    mongodbadapter
    mongo::mongocxx_shared
    mongo::bsoncxx_shared
    RTIConnextDDS::routing_service
    RTIConnextDDS::routing_service_infrastructure
    RTIConnextDDS::cpp2_api
    ${CONNEXTDDS_EXTERNAL_LIBS}
)

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/RsMongoGateway.xml"
    "${CMAKE_BINARY_DIR}/RsMongoGateway.xml"
    COPYONLY
)



#add_executable(
#    mongotest
#    "${CMAKE_BINARY_DIR}/ComplexType.cxx"
#    "${CMAKE_BINARY_DIR}/ComplexTypePlugin.cxx"
#    "${CMAKE_CURRENT_SOURCE_DIR}/SampleConverter.cxx"
#    "${CMAKE_CURRENT_SOURCE_DIR}/MongoTest.cpp")
#
#target_link_libraries(
#    mongotest
#    mongo::mongocxx_shared
#    mongo::bsoncxx_shared
#    RTIConnextDDS::cpp2_api
#    RTIConnextDDS::routing_service
#    ${CONNEXTDDS_EXTERNAL_LIBS}
#)
#
#target_include_directories(
#    mongotest
#    PUBLIC
#        "${CMAKE_BINARY_DIR}"
#)
#
#set_target_properties(mongotest
#    PROPERTIES
#    CXX_STANDARD 11
#    CXX_STANDARD_REQUIRED ON)