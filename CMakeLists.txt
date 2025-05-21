cmake_minimum_required(VERSION 3.15)
project(TradeMind VERSION 1.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# --- Find Dependencies ---
# Boost (example components, adjust as needed by the actual code)
find_package(Boost 1.70.0 COMPONENTS system thread REQUIRED)

# ZeroMQ
find_package(ZeroMQ REQUIRED)

# YAML-CPP
find_package(yaml-cpp 0.6 REQUIRED) # Version 0.6 is a common one, adjust if needed

# Fix8 (This is a common way to find it, but might vary based on Fix8 installation)
# You might need to provide hints for FIX8_ROOT or use a FindFIX8.cmake module
find_package(FIX8_LTD REQUIRED) # Assuming a modern CMake target `FIX8_LTD::FIX8_LTD`

# --- Include Directories ---
# Add the 'include' directory for project headers
include_directories(include)

# If Boost, ZeroMQ, yaml-cpp, Fix8 headers are not in standard paths,
# their include directories would be added automatically by find_package
# if they export CMake targets, or you might need to add them manually using
# target_include_directories after defining the target.

# --- Source Files ---
# List your C++ source files
# For now, we only see main.cpp. If there are other .cpp files in src/
# or its subdirectories, they need to be added here.
set(TRADEMIND_SOURCES
    src/main.cpp
    # Add other .cpp files here, e.g.:
    # src/core/FIXEngine.cpp
    # src/core/MessageBus.cpp
    # src/core/OrderBook.cpp
    # src/utils/Config.cpp
    # src/utils/Logger.cpp
)

# --- Define Executable Target ---
add_executable(trademind ${TRADEMIND_SOURCES})

# --- Link Libraries ---
target_link_libraries(trademind PRIVATE
    Boost::system
    Boost::thread
    # For ZeroMQ, the target name might be ZeroMQ::ZeroMQ, libzmq, or zmq depending on how it's found
    # Assuming modern CMake target:
    ZeroMQ::ZeroMQ
    # For yaml-cpp, target name is often yaml-cpp::yaml-cpp or just yaml-cpp
    yaml-cpp::yaml-cpp
    # For Fix8, if FIX8_LTD::FIX8_LTD is the target:
    FIX8_LTD::FIX8_LTD
)

# --- Output Directory for Executable ---
# The README implies running ./bin/trademind from the project root after building.
# This places the executable in the 'bin' directory at the project root.
set_target_properties(trademind PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin"
)

# Optional: If you have tests, you can enable CTest
# enable_testing()
# add_test(NAME MyTest COMMAND trademind --some-test-args)

# --- Installation (Optional, for deploying the application) ---
# install(TARGETS trademind DESTINATION bin)
# install(FILES config/config.yaml.example DESTINATION etc/trademind RENAME config.yaml)

message(STATUS "Configured TradeMind project.")
message(STATUS "  C++ Standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "  Boost found: ${Boost_FOUND}, Version: ${Boost_VERSION}")
message(STATUS "  ZeroMQ found: ${ZeroMQ_FOUND}")
message(STATUS "  yaml-cpp found: ${yaml-cpp_FOUND}")
message(STATUS "  FIX8_LTD found: ${FIX8_LTD_FOUND}")
message(STATUS "  Executable will be built in: ${CMAKE_SOURCE_DIR}/bin")