cmake_minimum_required(VERSION 3.26)

project(detector)

set(CMAKE_CXX_STANDARD 17)

option(ENABLE_HDF5 "Enable HDF5-based analysis output" ON)
option(BUILD_GOOGLE_TESTS "Build Google tests" OFF)

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
    if (NOT DEFINED yaml-cpp_DIR)
        set(yaml-cpp_DIR "E:/anaconda3/Library/lib/cmake/yaml-cpp" CACHE PATH "Path to yaml-cpp CMake config")
    endif()
endif()

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
    message("Windows system detected")
    include_directories("E:/Prophesee/third_party/include")
    link_directories("E:/Prophesee/third_party/lib" "E:/Prophesee/lib")

    set(BOOST_ROOT  "E:/Prophesee/third_party")
    set(Boost_INCLUDE_DIR "E:/Prophesee/third_party/include")
    set(Boost_LIBRARY_DIR "E:/Prophesee/third_party/lib")
    set(Boost_COMPILER "-vc140")
    set(Boost_ARCHITECTURE "-mt")

    set(TIFF_INCLUDE_DIR "E:/Prophesee/third_party/include")
    set(TIFF_LIBRARY "E:/Prophesee/third_party/lib")

    set(ZLIB_LIBRARY "E:/Prophesee/third_party/lib")
    set(ZLIB_INCLUDE_DIR "E:/Prophesee/third_party/include")

    set(Protobuf_INCLUDE_DIR "E:/Prophesee/third_party/include")
    set(Protobuf_LIBRARIES "E:/Prophesee/third_party/lib/libprotobuf.lib" "E:/Prophesee/third_party/lib/libprotoc.lib")

    #set(YAML_CPP_INCLUDE_DIR "c:/Users/radio/lib/yaml-cpp-0.8.0/include")
    #set(YAML_CPP_LIBRARY_DIR "c:/Users/radio/lib/yaml-cpp-0.8.0/lib")

    if (ENABLE_HDF5)
        set(HDF5_INCLUDE_DIRS "E:/Prophesee/third_party/include")
        set(HDF5_DIR "E:/Prophesee/third_party/share/hdf5")

        find_package(HDF5 CONFIG REQUIRED COMPONENTS C CXX)
    endif()
elseif (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    message("Linux system detected")
    add_definitions(-w -Wno-dev)

    if (ENABLE_HDF5)
        include_directories("/usr/include/hdf5/serial")
        find_package(HDF5 REQUIRED COMPONENTS C CXX)
    endif()

    include_directories("/usr/include/metavision")
    link_directories("/usr/lib/metavision")
endif()

find_package(yaml-cpp REQUIRED)
find_package(MetavisionSDK COMPONENTS base core driver ui REQUIRED)
find_package(Threads REQUIRED)
find_package(Boost COMPONENTS program_options REQUIRED)

set(EXECUTABLE qpdet)
set(CORRELATION_RESPONSE_MEASUREMENT_EXECUTABLE correlation_response_measurement)
set(PEAK_DETECTION_NO_TRIGGERS_EXECUTABLE peak_detection_no_triggers)

add_executable(${EXECUTABLE} main.cpp)

include_directories(QuadcopterPropellerDetector_lib)
add_subdirectory(QuadcopterPropellerDetector_lib)


set(LIBRARIES QuadcopterPropellerDetector_lib MetavisionSDK::base MetavisionSDK::core MetavisionSDK::driver MetavisionSDK::ui Boost::program_options yaml-cpp::yaml-cpp Threads::Threads)

if (ENABLE_HDF5)
    # if HDF5_LIBRARIES is not set, set it to the HDF5::HDF5 target
    if (NOT HDF5_LIBRARIES)
        set(HDF5_LIBRARIES hdf5 hdf5_cpp)
    endif()
    list(APPEND LIBRARIES ${HDF5_LIBRARIES})
endif()

target_link_libraries(${EXECUTABLE} ${LIBRARIES})

if (ENABLE_HDF5)
    target_compile_definitions(${EXECUTABLE} PRIVATE ENABLE_HDF5)
endif()

if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
    if (BUILD_IPU)
        target_link_libraries(${EXECUTABLE_IPU} QuadcopterPropellerDetector_lib MetavisionSDK::base MetavisionSDK::core MetavisionSDK::driver MetavisionSDK::ui Boost::program_options yaml-cpp Threads::Threads poplar poputil)
    endif()
endif()

if (CMAKE_SYSTEM_NAME STREQUAL "Windows" AND BUILD_GOOGLE_TESTS)
    add_subdirectory(Google_tests)
endif()
