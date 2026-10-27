cmake_minimum_required(VERSION 3.20 FATAL_ERROR)

# Enforce out-of-source builds.
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
    message(FATAL_ERROR "PandoraMonitoring requires an out-of-source build.")
endif()

project(PandoraMonitoring VERSION 05.02.00 LANGUAGES CXX)

# Set a default installation prefix to a local 'install' directory inside the build folder.
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/install" CACHE PATH "Default install path for PandoraMonitoring" FORCE)
endif()

if (NOT TARGET PandoraPFA::PandoraSDK)
    find_package(PandoraSDK 05.02.00 REQUIRED)
endif()
find_package(ROOT 6.18.04 REQUIRED COMPONENTS Eve Geom RGL EG)

# Build products

include(cmake/PandoraMonitoring_sources.cmake)

add_library(${PROJECT_NAME} SHARED ${PANDORA_MONITORING_SRCS})
add_library(PandoraPFA::PandoraMonitoring ALIAS ${PROJECT_NAME})

set_target_properties(${PROJECT_NAME} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
)

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

target_link_libraries(${PROJECT_NAME} PUBLIC
    PandoraPFA::PandoraSDK
    ROOT::Eve
    ROOT::Geom
    ROOT::RGL
    ROOT::EG
)

set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 17)
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD_REQUIRED ON)

target_compile_definitions(${PROJECT_NAME} PRIVATE -DMONITORING)
target_compile_options(${PROJECT_NAME} PRIVATE
    -Wall
    -Wextra
    -Werror
    -pedantic
    -Wno-long-long
    -Wno-sign-compare
    -Wshadow
    -fno-strict-aliasing
)

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(${PROJECT_NAME} PRIVATE -Wno-dtor-name)
endif()

# Optional documentation. 
option(PandoraMonitoring_BUILD_DOCS "Build documentation for ${PROJECT_NAME}" OFF)
if(PandoraMonitoring_BUILD_DOCS)
    add_subdirectory(doc)
endif()

# Installation

# Include standard CMake modules for generating package configuration files.
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

# Install the library target and export it for use by other CMake projects.
install(TARGETS ${PROJECT_NAME}
    EXPORT PandoraMonitoringTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Runtime
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT Development
    RUNTIME DESTINATION bin COMPONENT Runtime
)

# Install header files. 
install(DIRECTORY include/ DESTINATION include COMPONENT Development)

# Configure the main package file.
configure_package_config_file(
    "cmake/PandoraMonitoringConfig.cmake.in"
    "PandoraMonitoringConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PandoraMonitoring
)

# Generate the version file automatically.
write_basic_package_version_file(
    "PandoraMonitoringConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

# Install the generated package configuration files.
install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/PandoraMonitoringConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/PandoraMonitoringConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PandoraMonitoring
    COMPONENT Development
)

# Install the export file containing the target information.
install(EXPORT PandoraMonitoringTargets
    FILE PandoraMonitoringTargets.cmake
    NAMESPACE PandoraPFA::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/PandoraMonitoring
    COMPONENT Development
)
