cmake_minimum_required(VERSION 3.0.0)

list (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include (DetectCPPZMQVersion)

project(cppzmq VERSION ${DETECTED_CPPZMQ_VERSION})


if (EXISTS "${CMAKE_SOURCE_DIR}/.git")
    OPTION (ENABLE_DRAFTS "Build and install draft classes and methods" ON)
else ()
    OPTION (ENABLE_DRAFTS "Build and install draft classes and methods" OFF)
endif ()
if (ENABLE_DRAFTS)
    ADD_DEFINITIONS (-DZMQ_BUILD_DRAFT_API)
    set (pkg_config_defines "-DZMQ_BUILD_DRAFT_API=1")
else (ENABLE_DRAFTS)
    set (pkg_config_defines "")
endif (ENABLE_DRAFTS)

message(STATUS "cppzmq v${cppzmq_VERSION}")

set(CPPZMQ_HEADERS
    zmq.hpp
    zmq_addon.hpp
)

foreach (target cppzmq cppzmq-static)
  add_library(${target} INTERFACE)
  target_include_directories(${target} INTERFACE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                                                 $<INSTALL_INTERFACE:include>)
endforeach()

target_link_libraries(cppzmq INTERFACE libzmq)
target_link_libraries(cppzmq-static INTERFACE libzmq-static)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

install(TARGETS cppzmq cppzmq-static
        EXPORT ${PROJECT_NAME}-targets)

install(FILES ${CPPZMQ_HEADERS}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# GNUInstallDirs "DATADIR" wrong here; CMake search path wants "share".
set(CPPZMQ_CMAKECONFIG_INSTALL_DIR "share/cmake/${PROJECT_NAME}" CACHE STRING "install path for cppzmqConfig.cmake")


export(EXPORT ${PROJECT_NAME}-targets
     FILE "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake")
configure_package_config_file(${PROJECT_NAME}Config.cmake.in
                              "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
                              INSTALL_DESTINATION ${CPPZMQ_CMAKECONFIG_INSTALL_DIR})
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
                                 VERSION ${CPPZMQ_VERSION}
                                 COMPATIBILITY AnyNewerVersion)
install(EXPORT ${PROJECT_NAME}-targets
        FILE ${PROJECT_NAME}Targets.cmake
        DESTINATION ${CPPZMQ_CMAKECONFIG_INSTALL_DIR})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
              DESTINATION ${CPPZMQ_CMAKECONFIG_INSTALL_DIR})

option(CPPZMQ_BUILD_TESTS "Whether or not to build the tests" ON)

if (CPPZMQ_BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()
