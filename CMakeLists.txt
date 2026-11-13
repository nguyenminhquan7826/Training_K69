cmake_minimum_required(VERSION 3.16)
project(Training_K69 VERSION 0.1.0 LANGUAGES CXX)

# ---------------------------------------------------------------------------
# 1. Chuẩn C++ và thư mục output
# ---------------------------------------------------------------------------
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Sinh compile_commands.json cho IntelliSense / clangd
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Build mac dinh la Release neu nguoi dung khong chi dinh
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "" FORCE)
endif()

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# ---------------------------------------------------------------------------
# 2. Chan build tren he khong phai Linux
#    Source dung sys/socket.h, termios.h, pthread.h -> chi POSIX moi co
# ---------------------------------------------------------------------------
if(NOT UNIX OR APPLE)
    message(FATAL_ERROR
        "Du an nay dung POSIX API (sys/socket.h, termios.h, pthread.h) "
        "nen chi build duoc tren Linux (Raspberry Pi OS / Ubuntu). "
        "Tren Windows hay dung WSL2 hoac build truc tiep tren Pi.")
endif()

# ---------------------------------------------------------------------------
# 3. Tim thu vien phu thuoc
# ---------------------------------------------------------------------------
find_package(OpenCV REQUIRED)
find_package(Eigen3 3.3 REQUIRED NO_MODULE)
find_package(OsqpEigen REQUIRED)
find_package(Threads REQUIRED)

# libserial khong co CMake config -> dung pkg-config
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBSERIAL REQUIRED IMPORTED_TARGET libserial)

# ---------------------------------------------------------------------------
# 4. Thu vien loi (tat ca tru main.cpp)
#    Tach thanh library de sau nay them unit test dung lai duoc
# ---------------------------------------------------------------------------
add_library(k69_core
    src/LaneChangePlanner.cpp
    src/LaneDetector.cpp
    src/MpcController.cpp
    src/PurePursuitController.cpp
    src/Trans_UDP.cpp
    src/communication.cpp
    src/logger.cpp
    src/logic.cpp
)

# PUBLIC: moi target link toi k69_core deu thay duoc include/
target_include_directories(k69_core PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_link_libraries(k69_core PUBLIC
    ${OpenCV_LIBS}
    Eigen3::Eigen
    OsqpEigen::OsqpEigen
    PkgConfig::LIBSERIAL
    Threads::Threads
)

target_compile_options(k69_core PRIVATE
    -Wall -Wextra -Wpedantic -Wshadow
)

# ---------------------------------------------------------------------------
# 5. Chuong trinh chinh
# ---------------------------------------------------------------------------
add_executable(training_k69 src/main.cpp)
target_link_libraries(training_k69 PRIVATE k69_core)
# ---------------------------------------------------------------------------
# 6. Thong tin tom tat khi configure
# ---------------------------------------------------------------------------
message(STATUS "-------------------------------------------")
message(STATUS "Build type   : ${CMAKE_BUILD_TYPE}")
message(STATUS "OpenCV       : ${OpenCV_VERSION}")
message(STATUS "Eigen3       : ${Eigen3_VERSION}")
message(STATUS "libserial    : ${LIBSERIAL_VERSION}")
message(STATUS "Output       : ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/training_k69")
message(STATUS "-------------------------------------------")