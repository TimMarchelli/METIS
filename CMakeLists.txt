cmake_minimum_required(VERSION 3.22)

project(METIS C)

option(BUILD_SHARED_LIBS "Build shared libraries (.dll/.so) instead of static ones (.lib/.a)" ON)
option(METIS_IDX64 "enable 64 bit ints" OFF)
option(METIS_REAL64 "enable 64 bit floats (i.e., double)" On)
option(METIS_BUILD_PROGRAMS "Build METIS programs" Off)

# export all symbols
if(BUILD_SHARED_LIBS)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS TRUE)
endif()

# get GKlib
find_package(GKlib REQUIRED)

# set a debug postfix
set(CMAKE_DEBUG_POSTFIX "d")

# default install location vars and configuration build tool
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# modify install targets
set(INSTALL_RUNTIME_DIR ${CMAKE_INSTALL_BINDIR})
set(INSTALL_CONFIG_DIR  ${CMAKE_INSTALL_LIBDIR}/cmake/METIS)
set(INSTALL_LIBRARY_DIR ${CMAKE_INSTALL_LIBDIR})
set(INSTALL_INCLUDE_DIR ${CMAKE_INSTALL_INCLUDEDIR})

# rpath
if(NOT APPLE)
    set(CMAKE_INSTALL_RPATH $ORIGIN)
endif()

# include the compiler option macro
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/ProjectCompilerSettings.cmake)

# setup GKlib options
include(${CMAKE_CURRENT_SOURCE_DIR}/conf/gkbuild.cmake)

# set source files
file(GLOB metis_sources ${CMAKE_CURRENT_SOURCE_DIR}/libmetis/*.c)

# set public headers
file(GLOB metis_public_headers ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)

# add metis library
add_library(metis ${metis_sources})

# set C standard to C99
set_target_properties(metis PROPERTIES C_STANDARD 99)

# set compiler options
target_compile_options(metis PRIVATE
    $<$<C_COMPILER_ID:GNU>:-fno-strict-aliasing>
    $<$<AND:$<PLATFORM_ID:Linux>,$<C_COMPILER_ID:GNU>>:-fPIC>
)

# set common options and warnings
ProjectCompilerSettings(metis)

# METIS' custom options
if(IDX64)
    target_compile_definitions(metis PUBLIC IDXTYPEWIDTH=64)
else()
    target_compile_definitions(metis PUBLIC IDXTYPEWIDTH=32)
endif()

if(REAL64)
    target_compile_definitions(metis PUBLIC REALTYPEWIDTH=64)
else()
    target_compile_definitions(metis PUBLIC REALTYPEWIDTH=32)
endif()

# add includes
target_include_directories(metis 
    PUBLIC 
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${INSTALL_INCLUDEDIR}>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/libmetis
)

# link dependencies
target_link_libraries(metis PRIVATE GKlib::GKlib)

# add an alias for find
add_library(METIS::metis ALIAS metis)

# add version information
set_target_properties(metis PROPERTIES 
    SOVERSION 5
    VERSION   5.2.1
)

if(METIS_BUILD_PROGRAMS)
    # include CTest here so tests appear
    include(CTest)
    
    add_subdirectory(programs)
endif()

# set a project config file to enable linking to libraries in the build directory
set(PROJECT_CONFIG_FILE "${CMAKE_BINARY_DIR}/METISConfig.cmake")
configure_package_config_file(METISConfig.cmake.in
    ${PROJECT_CONFIG_FILE}
    INSTALL_DESTINATION ${INSTALL_CONFIG_DIR}
)

# install library
install(TARGETS metis
    EXPORT METIS-targets
    RUNTIME DESTINATION ${INSTALL_RUNTIME_DIR}
    LIBRARY DESTINATION ${INSTALL_LIBRARY_DIR}
    ARCHIVE DESTINATION ${INSTALL_LIBRARY_DIR}
)

# install includes
install(FILES ${metis_public_headers} DESTINATION ${INSTALL_INCLUDE_DIR})

install(FILES
    ${PROJECT_CONFIG_FILE}
    DESTINATION ${INSTALL_CONFIG_DIR}
)

# install config
install(EXPORT METIS-targets 
    FILE METIS-targets.cmake 
    NAMESPACE METIS::
    DESTINATION ${INSTALL_CONFIG_DIR}
)
