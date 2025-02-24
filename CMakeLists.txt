include(CheckLibraryExists)
include(CheckIncludeFile)
include(CheckSymbolExists)
include(ExternalProject)

# Adds the cmake directory to the CMake include path.
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

# X_VCPKG_APPLOCAL_DEPS_INSTALL automatically installs dependencies. CMP0077
# supress some arcane CMake warning.
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)
set(X_VCPKG_APPLOCAL_DEPS_INSTALL ON)

cmake_minimum_required(VERSION 3.6)

project("Woof"
    VERSION 10.5.1
    LANGUAGES C)

set(CMAKE_C_STANDARD 99)

# Set a default build type if none was specified
set(default_build_type "RelWithDebInfo")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# Prevent in-tree builds.
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
    message(FATAL_ERROR "In-tree builds are not supported.")
endif()

execute_process(COMMAND uname OUTPUT_VARIABLE uname)
if (NOT MSVC AND (uname MATCHES "^MSYS" OR uname MATCHES "^MINGW"))
    set(MSYS ON)
else()
    set(MSYS OFF)
endif()

# Hardcoded defines added to configure and resource files.
set(PROJECT_COMPANY "Fabian Greffrath and contributors")
set(PROJECT_COPYRIGHT "Copyright (C) 1993-2021")
set(PROJECT_LICENSE "GNU General Public License, version 2")
set(PROJECT_STRING "${PROJECT_NAME} ${PROJECT_VERSION}")
set(PROJECT_SHORTNAME "woof")
set(WOOF_ICON "woof.ico")
set(SETUP_ICON "setup.ico")
set(PROJECT_VERSION_RC "${PROJECT_VERSION_MAJOR},${PROJECT_VERSION_MINOR},${PROJECT_VERSION_PATCH},0") # ${PROJECT_VERSION_TWEAK}

if(NOT WIN32)
    set(WOOFDATADIR "${CMAKE_INSTALL_PREFIX}/share/${PROJECT_SHORTNAME}" CACHE PATH "Path to install supplemental files")
endif()

# Toggle-able defines added at compile-time.
option("${PROJECT_NAME}_RANGECHECK" "Enable bounds-checking of performance-sensitive functions" ON)
option("${PROJECT_NAME}_STRICT" "Prefer original MBF code paths over demo compatiblity with PrBoom+" OFF)

# Compiler environment requirements.
check_library_exists(m pow "" HAVE_LIBM)
check_include_file("dirent.h" HAVE_DIRENT_H)
check_symbol_exists(strsignal "string.h" HAVE_STRSIGNAL)
check_symbol_exists(strcasecmp "strings.h" HAVE_DECL_STRCASECMP)
check_symbol_exists(strncasecmp "strings.h" HAVE_DECL_STRNCASECMP)

option(CMAKE_FIND_PACKAGE_PREFER_CONFIG
       "Lookup package config files before using find modules" ON)

# Library requirements.
find_package(SDL2 2.0.7 REQUIRED)
find_package(SDL2_mixer 2.0.2 REQUIRED)
find_package(SDL2_net REQUIRED)

find_package(FluidSynth 2.2.0)
find_package(SndFile 1.0.28)

# Python 3
find_package(Python3 COMPONENTS Interpreter)

configure_file(config.h.in config.h)

if(WIN32)
    install(FILES COPYING DESTINATION .)
    install(FILES README.md DESTINATION .)
    install(DIRECTORY docs/ DESTINATION docs)
    install(DIRECTORY examples/ DESTINATION docs/examples)
    install(DIRECTORY autoload/ DESTINATION autoload)
    if(FluidSynth_FOUND)
        install(DIRECTORY soundfonts/ DESTINATION soundfonts)
    endif()
else()
    install(FILES COPYING DESTINATION "share/doc/${PROJECT_SHORTNAME}")
    install(FILES README.md DESTINATION "share/doc/${PROJECT_SHORTNAME}")
    install(DIRECTORY autoload/ DESTINATION "share/${PROJECT_SHORTNAME}/autoload")
    if(EXISTS "${CMAKE_SOURCE_DIR}/docs")
        install(DIRECTORY docs/ DESTINATION "share/doc/${PROJECT_SHORTNAME}")
    endif()
    if(EXISTS "${CMAKE_SOURCE_DIR}/examples")
        install(DIRECTORY examples/ DESTINATION "share/doc/${PROJECT_SHORTNAME}/examples")
    endif()
    if(EXISTS "${CMAKE_SOURCE_DIR}/soundfonts" AND FluidSynth_FOUND)
        install(DIRECTORY soundfonts/ DESTINATION "share/${PROJECT_SHORTNAME}/soundfonts")
    endif()
endif()

# Generate distribution packages with CPack.
if(WIN32)
    set(CPACK_GENERATOR ZIP)
else()
    set(CPACK_GENERATOR TGZ)
endif()
set(CPACK_SOURCE_GENERATOR TGZ ZIP)
set(CPACK_SOURCE_IGNORE_FILES "/.git/;/build;/.vs/;/out/;CMakeSettings.json")
set(CPACK_STRIP_FILES TRUE)
include(CPack)

# Where to find other CMakeLists.txt files.
add_subdirectory(data)
add_subdirectory(opl)
add_subdirectory(textscreen)
add_subdirectory(src)
add_subdirectory(toolsrc)
add_subdirectory(setup)
add_subdirectory(man)
