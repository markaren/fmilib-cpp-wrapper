
cmake_minimum_required(VERSION 3.9)
file(STRINGS "${CMAKE_SOURCE_DIR}/version.txt" projectVersion)
project("fmilibcpp" VERSION ${projectVersion})
message("Current ${PROJECT_NAME} version: ${CMAKE_PROJECT_VERSION}\n")

# ==============================================================================
# Build settings
# ==============================================================================

option(BUILD_SHARED_LIBS OFF)
option(FMILIBCPP_BUILD_TESTS "Build tests" OFF)
option(FMILIBCPP_BUILD_EXAMPLES "Build examples" OFF)

# ==============================================================================
# Global internal configuration
# ==============================================================================

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

if (MSVC)
  set(CMAKE_DEBUG_POSTFIX "d")
  add_definitions("-D_WIN32_WINNT=0x600" "-D_CRT_SECURE_NO_WARNINGS")
  add_compile_options("/wd4996" "/wd4251" "/wd4244" "/wd4267")
endif ()

# Prepare for export and installation.
set(FMILIBCPP_HEADER_INSTALL_DIR "include")
if (WIN32)
  set(FMILIBCPP_CMAKE_INSTALL_DIR "cmake")
else ()
  set(FMILIBCPP_CMAKE_INSTALL_DIR "share/${PROJECT_NAME}/cmake")
endif ()

set(FMILIBCPP_INSTALL_DESTINATIONS
        ARCHIVE DESTINATION "lib"
        LIBRARY DESTINATION "lib"
        RUNTIME DESTINATION "bin"
        INCLUDES DESTINATION "${FMILIBCPP_HEADER_INSTALL_DIR}")
set(FMILIBCPP_EXPORT_TARGET "${PROJECT_NAME}-targets")


# ==============================================================================
# Dependencies
# ==============================================================================

if (EXISTS ${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
  include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
  conan_basic_setup()
else ()
  message(FATAL_ERROR "The file conanbuildinfo.cmake doesn't exist, you have to run conan install first!")
endif ()

if (FMILIBCPP_BUILD_TESTS)
  set(Boost_USE_STATIC_LIBS ON)
  find_package(Boost 1.66 COMPONENTS unit_test_framework REQUIRED)
endif ()

find_package(FMILIB REQUIRED)

# ==============================================================================
# Targets
# ==============================================================================

add_subdirectory(src)

if (FMILIBCPP_BUILD_EXAMPLES OR FMILIBCPP_BUILD_TESTS)
  file(COPY data/fmus DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

  if (FMILIBCPP_BUILD_EXAMPLES)
    add_subdirectory(examples)
  endif ()

  if (FMILIBCPP_BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
  endif ()

endif ()


# ==============================================================================
# Exports and remaining installation
# ==============================================================================

install(
        EXPORT "${FMILIBCPP_EXPORT_TARGET}"
        DESTINATION "${FMILIBCPP_CMAKE_INSTALL_DIR}"
        NAMESPACE "${PROJECT_NAME}::"
)

include(CMakePackageConfigHelpers)

# Generate and install package-config file.
set(configFile "${CMAKE_BINARY_DIR}/${PROJECT_NAME}-config.cmake")
set(targetsFile "${FMILIBCPP_CMAKE_INSTALL_DIR}/${FMILIBCPP_EXPORT_TARGET}.cmake")
configure_package_config_file(
        "${CMAKE_SOURCE_DIR}/cmake/project-config.cmake.in"
        "${configFile}"
        INSTALL_DESTINATION "${FMILIBCPP_CMAKE_INSTALL_DIR}"
        PATH_VARS targetsFile
)
install(FILES "${configFile}" DESTINATION "${FMILIBCPP_CMAKE_INSTALL_DIR}")

# Generate and install package-version file
set(versionFile "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}-config-version.cmake")
write_basic_package_version_file(
        "${versionFile}"
        VERSION "${PROJECT_VERSION}"
        COMPATIBILITY "SameMajorVersion")
install(FILES "${versionFile}" DESTINATION "${FMILIBCPP_CMAKE_INSTALL_DIR}")

# Install custom find modules
install(FILES
        "${CMAKE_SOURCE_DIR}/cmake/FindFMILIB.cmake"
        DESTINATION
        "${FMILIBCPP_CMAKE_INSTALL_DIR}"
        )
