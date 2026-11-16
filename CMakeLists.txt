cmake_minimum_required(VERSION 3.21)
if ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS)
    set(CMAKE_OSX_ARCHITECTURES "arm64")
else()
    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")
endif()
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(BasuInputBuffer VERSION 2.9.0)

file(GLOB SOURCES CONFIGURE_DEPENDS src/*.cpp)
add_library(${PROJECT_NAME} SHARED ${SOURCES})

if (NOT DEFINED ENV{GEODE_SDK})
    message(FATAL_ERROR "Unable to find Geode SDK! Please define GEODE_SDK environment variable to point to Geode")
endif()

add_subdirectory($ENV{GEODE_SDK} ${CMAKE_CURRENT_BINARY_DIR}/geode)

# iOS: Geode's generated bindings source uses FMOD types without including the FMOD header
# (fixed in the bindings repo later); force-include it so the iOS build compiles
if (("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS) AND TARGET GeodeBindings)
    target_compile_options(GeodeBindings PRIVATE -include Geode/fmod/fmod.hpp)
endif()

# iOS: no extra signing step (v3.0.4). The v2.99.2 ad-hoc "codesign --force --sign -" made the
# Geode iOS launcher's JIT-less signer leave the file alone, so iOS rejected it ("code signature
# invalid"). Built like every other Geode mod, the launcher signs it with your certificate.

setup_geode_mod(${PROJECT_NAME})
