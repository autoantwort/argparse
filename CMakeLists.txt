cmake_minimum_required(VERSION 3.12.4)

if(NOT DEFINED PROJECT_NAME)
  set(ARGPARSE_IS_TOP_LEVEL ON)
else()
  set(ARGPARSE_IS_TOP_LEVEL OFF)
endif()

if(${CMAKE_VERSION} VERSION_GREATER_EQUAL "4.0.0")
    set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "d0edc3af-4c50-42ea-a356-e2862fe7a444") # 4.0.3
else()
    set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD "0e5b6991-d74f-4b3d-a41c-cf096e0b2508") # 3.31.0
endif()

project(argparse
        VERSION 3.2.0 
        DESCRIPTION "A single header argument parser for C++17"
        HOMEPAGE_URL "https://github.com/p-ranav/argparse"
        LANGUAGES CXX
)

option(ARGPARSE_INSTALL "Include an install target" ${ARGPARSE_IS_TOP_LEVEL})
option(ARGPARSE_BUILD_TESTS "Build tests" ${ARGPARSE_IS_TOP_LEVEL})
option(ARGPARSE_BUILD_SAMPLES "Build samples" OFF)
option(ARGPARSE_EXPORT_MODULE "Exports an c++20 module" OFF)
option(ARGPARSE_USE_IMPORT_STD "Use import std;" ON)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

add_library(argparse INTERFACE)
add_library(argparse::argparse ALIAS argparse)

target_compile_features(argparse INTERFACE cxx_std_17)
target_include_directories(argparse INTERFACE
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>)

if(ARGPARSE_BUILD_SAMPLES)
  add_subdirectory(samples)
endif()
  
if(ARGPARSE_BUILD_TESTS)
  add_subdirectory(test)
endif()

if (ARGPARSE_EXPORT_MODULE)
  add_library(argparse-modules)
  add_library(argparse::argparse-modules ALIAS argparse-modules)
  target_link_libraries(argparse-modules PRIVATE argparse)
  target_compile_features(argparse-modules PUBLIC cxx_std_20)
  target_sources(argparse-modules PUBLIC FILE_SET CXX_MODULES FILES module/argparse.cppm)
  if (ARGPARSE_USE_IMPORT_STD)
    target_compile_definitions(argparse-modules PRIVATE ARGPARSE_MODULE_USE_STD_MODULE)
    set_target_properties(argparse-modules PROPERTIES CXX_MODULE_STD ON)
    target_compile_features(argparse-modules PRIVATE cxx_std_23)
  endif()
endif()
  
if(ARGPARSE_INSTALL)
  install(TARGETS argparse EXPORT argparseConfig)
  if (ARGPARSE_EXPORT_MODULE)
    install(TARGETS argparse-modules EXPORT argparseConfig FILE_SET CXX_MODULES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/modules")
  endif()
  install(EXPORT argparseConfig
          NAMESPACE argparse::
          DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
  install(FILES ${CMAKE_CURRENT_LIST_DIR}/include/argparse/argparse.hpp
          DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/argparse)


  set(CONFIG_FILE_NAME_WITHOUT_EXT "${PROJECT_NAME}Config")
  set(CMAKE_CONFIG_FILE_BASENAME "${CMAKE_CURRENT_BINARY_DIR}/${CONFIG_FILE_NAME_WITHOUT_EXT}")
  set(CMAKE_CONFIG_VERSION_FILE_NAME "${CMAKE_CONFIG_FILE_BASENAME}-version.cmake")
  set(CMAKE_CONFIG_FILE_NAME "${CMAKE_CONFIG_FILE_BASENAME}.cmake")

  if(${CMAKE_VERSION} VERSION_GREATER "3.14")
  	set(OPTIONAL_ARCH_INDEPENDENT "ARCH_INDEPENDENT")
  endif()

  write_basic_package_version_file("${CMAKE_CONFIG_VERSION_FILE_NAME}"
      COMPATIBILITY SameMajorVersion
      ${OPTIONAL_ARCH_INDEPENDENT}
  )

  export(EXPORT argparseConfig
         NAMESPACE argparse::)

  install(FILES "${CMAKE_CONFIG_VERSION_FILE_NAME}"
         DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

  set(PackagingTemplatesDir "${CMAKE_CURRENT_SOURCE_DIR}/packaging")

  set(CPACK_PACKAGE_NAME "${PROJECT_NAME}")
  set(CPACK_PACKAGE_VENDOR "argparse (C++) developers")
  set(CPACK_PACKAGE_DESCRIPTION "${PROJECT_DESCRIPTION}")
  set(CPACK_DEBIAN_PACKAGE_NAME "${CPACK_PACKAGE_NAME}")
  set(CPACK_RPM_PACKAGE_NAME "${CPACK_PACKAGE_NAME}")
  set(CPACK_PACKAGE_HOMEPAGE_URL "${PROJECT_HOMEPAGE_URL}")
  set(CPACK_PACKAGE_MAINTAINER "Pranav Srinivas Kumar")
  set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_MAINTAINER}")
  set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
  set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

  set(CPACK_DEBIAN_PACKAGE_NAME "lib${PROJECT_NAME}-dev")
  set(CPACK_DEBIAN_PACKAGE_DEPENDS "libc6-dev")
  set(CPACK_DEBIAN_PACKAGE_SUGGESTS "cmake, pkg-config, pkg-conf")

  set(CPACK_RPM_PACKAGE_NAME "lib${PROJECT_NAME}-devel")
  set(CPACK_RPM_PACKAGE_SUGGESTS "${CPACK_DEBIAN_PACKAGE_SUGGESTS}")

  set(CPACK_DEB_COMPONENT_INSTALL ON)
  set(CPACK_RPM_COMPONENT_INSTALL ON)
  set(CPACK_NSIS_COMPONENT_INSTALL ON)
  set(CPACK_DEBIAN_COMPRESSION_TYPE "xz")

  set(PKG_CONFIG_FILE_NAME "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc")
  configure_file("${PackagingTemplatesDir}/pkgconfig.pc.in" "${PKG_CONFIG_FILE_NAME}" @ONLY)
  install(FILES "${PKG_CONFIG_FILE_NAME}"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
  )
endif()

include(CPack)
