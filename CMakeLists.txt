cmake_minimum_required(VERSION 2.8)

cmake_policy(SET CMP0042 NEW)

option(DUCKDB_WASM_EXTENSIONS "Whether compiling for Wasm target" OFF)

###
# Configuration
###
if(NOT DEFINED MINIMUM_DUCKDB_VERSION_MAJOR OR
        NOT DEFINED MINIMUM_DUCKDB_VERSION_MINOR OR
        NOT DEFINED MINIMUM_DUCKDB_VERSION_PATCH)
    message(FATAL_ERROR "DuckDB minimum version is required")
endif()
if(NOT DEFINED EXTENSION_NAME)
    message(FATAL_ERROR "DuckDB extension name is required")
endif()
add_definitions(-DDUCKDB_EXTENSION_API_VERSION_MAJOR=${MINIMUM_DUCKDB_VERSION_MAJOR})
add_definitions(-DDUCKDB_EXTENSION_API_VERSION_MINOR=${MINIMUM_DUCKDB_VERSION_MINOR})
add_definitions(-DDUCKDB_EXTENSION_API_VERSION_PATCH=${MINIMUM_DUCKDB_VERSION_PATCH})
add_definitions(-DDUCKDB_EXTENSION_NAME=${EXTENSION_NAME})

###
# Build
###
project(${EXTENSION_NAME} LANGUAGES C)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Include own headers
include_directories(src/include)
# Include DuckDB C API headers
include_directories(duckdb_capi)

add_subdirectory(src)

# Create Extension library
set(EXTENSION_SOURCES
        ${ALL_OBJECT_FILES}
        src/main.c
)

if (DUCKDB_WASM_EXTENSION)
	add_library(${EXTENSION_NAME} STATIC ${EXTENSION_SOURCES})
else()
	add_library(${EXTENSION_NAME} SHARED ${EXTENSION_SOURCES})
endif()

# Hide symbols
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_C_VISIBILITY_PRESET hidden)
set(VISIBILITY_INLINES_HIDDEN ON)
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -s")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -s")