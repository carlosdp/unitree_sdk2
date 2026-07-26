cmake_minimum_required(VERSION 3.12)
project(unitree_interface_python)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(pybind11 REQUIRED)

# Use the unitree_sdk2 target from parent project
# This automatically provides all necessary include paths and libraries
if(NOT TARGET unitree_sdk2)
    message(FATAL_ERROR "unitree_sdk2 target not found. Make sure this is built from the parent project.")
endif()

# Add source files for general interface
set(GENERAL_SOURCES
    unitree_interface.cpp
    unitree_interface_bindings.cpp
)

# Create pybind11 module
pybind11_add_module(unitree_interface ${GENERAL_SOURCES})

# Compiler-specific options
target_compile_definitions(unitree_interface PRIVATE VERSION_INFO=${PROJECT_VERSION})

# Link with unitree_sdk2 which provides all necessary dependencies
target_link_libraries(unitree_interface PRIVATE unitree_sdk2)

# Ensure runtime can find bundled DDS libraries when loaded from a wheel
set_target_properties(unitree_interface PROPERTIES
    BUILD_RPATH "$ORIGIN"
    INSTALL_RPATH "$ORIGIN"
)

# Bundle DDS shared libraries next to the Python extension so they are
# available at runtime without system-wide installs.
set(DDS_LIB_DIR ${CMAKE_SOURCE_DIR}/thirdparty/lib/${CMAKE_SYSTEM_PROCESSOR})
set(DDS_LIBS
    ${DDS_LIB_DIR}/libddsc.so
    ${DDS_LIB_DIR}/libddscxx.so
)

foreach(lib_file IN LISTS DDS_LIBS)
    if(NOT EXISTS ${lib_file})
        message(FATAL_ERROR "Missing required DDS runtime library: ${lib_file}")
    endif()
endforeach()

install(FILES ${DDS_LIBS} DESTINATION .)

# Create SONAME-compatible symlinks inside the wheel install directory so the
# Python extension can resolve libddsc.so.0 and libddscxx.so.0 at runtime.
install(CODE [[
    if(EXISTS "${CMAKE_INSTALL_PREFIX}/libddsc.so")
        execute_process(COMMAND ${CMAKE_COMMAND} -E remove -f "${CMAKE_INSTALL_PREFIX}/libddsc.so.0")
        execute_process(
            COMMAND ${CMAKE_COMMAND} -E create_symlink libddsc.so libddsc.so.0
            WORKING_DIRECTORY "${CMAKE_INSTALL_PREFIX}"
        )
    endif()
    if(EXISTS "${CMAKE_INSTALL_PREFIX}/libddscxx.so")
        execute_process(COMMAND ${CMAKE_COMMAND} -E remove -f "${CMAKE_INSTALL_PREFIX}/libddscxx.so.0")
        execute_process(
            COMMAND ${CMAKE_COMMAND} -E create_symlink libddscxx.so libddscxx.so.0
            WORKING_DIRECTORY "${CMAKE_INSTALL_PREFIX}"
        )
    endif()
]])

# Enable compiler warnings
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(unitree_interface PRIVATE -Wall -Wextra)
endif()

# Optional: Generate stub files for type hints (disabled by default because
# the inline Python snippet breaks Ninja when enabled inside scikit-build)
option(GENERATE_STUBS "Generate Python stub files" OFF)

if(GENERATE_STUBS)
    # Add custom target to generate stub files
    add_custom_target(generate_stubs
        COMMAND ${CMAKE_COMMAND} -E echo "Generating Python stub files..."
        COMMAND python3 -c "
import sys
sys.path.insert(0, '${CMAKE_CURRENT_BINARY_DIR}')
try:
    import unitree_interface
    import pybind11_stubgen
    pybind11_stubgen.main(['-o', '${CMAKE_CURRENT_BINARY_DIR}', 'unitree_interface'])
    print('Stub files generated successfully')
except ImportError as e:
    print(f'Warning: Could not generate stub files: {e}')
    print('Install pybind11-stubgen: pip install pybind11-stubgen')
except Exception as e:
    print(f'Error generating stubs: {e}')
"
        DEPENDS unitree_interface
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generating Python stub files"
    )
endif()

# Installation
install(TARGETS unitree_interface DESTINATION .)

# Print configuration summary
message(STATUS "=== Unitree Interface Python Binding Configuration ===")
message(STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
message(STATUS "Using unitree_sdk2 target from parent project")
message(STATUS "GENERATE_STUBS: ${GENERATE_STUBS}")
message(STATUS "Modules to build:")
message(STATUS "  - unitree_interface (general interface for all robot types)")
message(STATUS "===================================================")
