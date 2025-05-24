#set( CMAKE_CXX_COMPILER "C:\msys64\ucrt64\bin\g++.exe" )
#set( CMAKE_C_COMPILER "C:\msys64\ucrt64\bin\gcc.exe" )

cmake_minimum_required(VERSION 3.15)
project(TradeMind VERSION 1.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
find_package(PkgConfig REQUIRED)

# --- Find Dependencies ---
# Boost (example components, adjust as needed by the actual code)
find_package(Boost 1.70.0 COMPONENTS system thread REQUIRED)

# ZeroMQ
pkg_check_modules(PC_ZMQ REQUIRED libzmq)

# YAML-CPP
find_package(yaml-cpp 0.6 REQUIRED) # Version 0.6 is a common one, adjust if needed

# Fix8 (This is a common way to find it, but might vary based on Fix8 installation)
# You might need to provide hints for FIX8_ROOT or use a FindFIX8.cmake module
find_package(Fix8) # Assuming a modern CMake target `FIX8_LTD::FIX8_LTD`


set(TRADEMIND_SOURCES
    src/main.cpp
    # Add other .cpp files here
)

# --- Define Executable Target ---
# THIS MUST COME BEFORE ANY target_*(trademind ...) commands
add_executable(trademind ${TRADEMIND_SOURCES})

# --- Include Directories ---
# Add the project's own 'include' directory first
target_include_directories(trademind PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
)



# --- Include Directories ---
# Add the 'include' directory for project headers
include_directories(include)

if(PC_ZMQ_FOUND) # This 'if' is technically redundant due to REQUIRED, but good for structure
    target_include_directories(trademind PRIVATE ${PC_ZMQ_INCLUDE_DIRS})
endif()


if(Fix8_FOUND)
    message(STATUS "Found Fix8 version ${FIX8_VERSION_STRING}. Adding Fix8 include directory: ${FIX8_INCLUDE_DIRS}")
    # The FindFix8.cmake sets FIX8_INCLUDE_DIRS
    target_include_directories(trademind PRIVATE ${FIX8_INCLUDE_DIRS})
else()
    message(STATUS "Fix8 not found. Fix8-related features might be disabled or build might fail later if code uses it.")
endif()
# If Boost, ZeroMQ, yaml-cpp, Fix8 headers are not in standard paths,
# their include directories would be added automatically by find_package
# if they export CMake targets, or you might need to add them manually using
# target_include_directories after defining the target.

# --- Link Libraries ---
target_link_libraries(trademind PRIVATE
    Boost::system
    Boost::thread
    # For ZeroMQ, the target name might be ZeroMQ::ZeroMQ, libzmq, or zmq depending on how it's found
    # Assuming modern CMake target:
    ${PC_ZMQ_LIBRARIES}
    # For yaml-cpp, target name is often yaml-cpp::yaml-cpp or just yaml-cpp
    yaml-cpp::yaml-cpp
)


if(Fix8_FOUND)
    # If FindFix8.cmake was expected to create an imported target like Fix8::Fix8 or FIX8_LTD::FIX8_LTD
    # target_link_libraries(trademind PRIVATE Fix8::Fix8) # Example if it creates this target
    # If it just sets FIX8_LIBRARIES (as our FindFix8.cmake seems to do):
    target_link_libraries(trademind PRIVATE ${FIX8_LIBRARIES})
endif()

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