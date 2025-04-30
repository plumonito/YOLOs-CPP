cmake_minimum_required(VERSION 3.14.0)
project(yolos_cpp LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(ONNXRUNTIME_DIR "Path to built ONNX Runtime directory." STRING)
message(STATUS "ONNXRUNTIME_DIR: ${ONNXRUNTIME_DIR}")

find_package(OpenCV REQUIRED)

# Create an import target for onnxruntime
add_library(onnxruntime::onnxruntime SHARED IMPORTED)
target_include_directories(onnxruntime::onnxruntime INTERFACE "${ONNXRUNTIME_DIR}/include")

if(UNIX)
    message(STATUS "We are building on Linux!")
    # Specific Linux build commands or flags
    set(ONNXLIBNAME "libonnxruntime.so")
endif(UNIX)

if(APPLE)
    message(STATUS "We are building on macOS!")
    # Specific macOS build commands or flags
    set(ONNXLIBNAME "libonnxruntime.dylib")
endif(APPLE)

if(WIN32)
    message(STATUS "We are building on Windows!")
    # Specific Windows build commands or flags
    set(ONNXLIBNAME "onnxruntime.lib")
endif(WIN32)
set_target_properties(onnxruntime::onnxruntime PROPERTIES IMPORTED_LOCATION "${ONNXRUNTIME_DIR}/lib/${ONNXLIBNAME}")


# Create the header-only yolos-cpp target
file(GLOB_RECURSE YOLOS_CPP_HEADERS include/*.hpp)
add_library(yolos_cpp INTERFACE ${YOLOS_CPP_HEADERS})

target_include_directories(yolos_cpp INTERFACE "include/")
target_compile_features(yolos_cpp INTERFACE cxx_std_14)
target_link_libraries(yolos_cpp INTERFACE onnxruntime::onnxruntime) # onnxruntime is a core dependency
target_link_libraries(yolos_cpp INTERFACE opencv_core opencv_imgproc) # only core and imgproc are needed by the headers

# Add executable for image inference
add_executable(image_inference
               src/image_inference.cpp)

# Add executable for camera inference
add_executable(camera_inference
               src/camera_inference.cpp)

# Add executable for video inference
add_executable(video_inference
               src/video_inference.cpp)

# Link libraries for all executables
# opencv_highgui is needed by the examples to load and display images
target_link_libraries(image_inference yolos_cpp opencv_highgui)
target_link_libraries(camera_inference yolos_cpp opencv_highgui)
target_link_libraries(video_inference yolos_cpp opencv_highgui opencv_videoio)
