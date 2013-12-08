cmake_minimum_required ( VERSION 2.8 )

project(libmpsse)
set(MAJOR_VERSION 1)
set(MINOR_VERSION 0)
set(PACKAGE libmpsse)
set(VERSION_STRING ${MAJOR_VERSION}.${MINOR_VERSION})
set(VERSION ${VERSION_STRING})

# find libftdi
find_package ( LibFTDI1 NO_MODULE REQUIRED )
include ( ${LIBFTDI_USE_FILE} )

# PkgConfig
set(prefix      ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX}/bin)
set(includedir  ${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME})

# guess LIB_SUFFIX, don't take debian multiarch into account
if ( NOT DEFINED LIB_SUFFIX )
  if( CMAKE_SYSTEM_NAME MATCHES "Linux"
      AND NOT CMAKE_CROSSCOMPILING
      AND NOT EXISTS "/etc/debian_version"
      AND NOT EXISTS "/etc/arch-release" )
    if ( "${CMAKE_SIZEOF_VOID_P}" EQUAL "8" )
      set ( LIB_SUFFIX 64 )
    endif ()
  endif ()
endif ()

# NOTE: src/CMakeLists.txt uses ${LIB_SUFFIX}
add_subdirectory(src)

if ( UNIX )
  set(libdir	   ${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX})

  configure_file ( libmpsse-config.in ${CMAKE_CURRENT_BINARY_DIR}/libmpsse-config @ONLY )
  install ( PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/libmpsse-config
            DESTINATION bin )

  # config script install path
  if ( NOT DEFINED LIBMPSSE_CMAKE_CONFIG_DIR )
    set ( LIBMPSSE_CMAKE_CONFIG_DIR lib${LIB_SUFFIX}/cmake/libmpsse )
  endif ()

  set ( LIBMPSSE_INCLUDE_DIR ${includedir} )
  set ( LIBMPSSE_INCLUDE_DIRS ${LIBMPSSE_INCLUDE_DIR} )
  list ( APPEND LIBMPSSE_INCLUDE_DIRS ${LIBFTDI_INCLUDE_DIRS} )
  set (LIBMPSSE_LIBRARY mpsse)
  set ( LIBMPSSE_LIBRARIES ${LIBMPSSE_LIBRARY} )
  list ( APPEND LIBMPSSE_LIBRARIES ${LIBFTDI_LIBRARIES} )
  set ( LIBMPSSE_LIBRARY_DIRS ${libdir} )
  set ( LIBMPSSE_ROOT_DIR ${prefix} )
  set ( LIBMPSSE_VERSION_STRING ${VERSION_STRING} )
  set ( LIBMPSSE_VERSION_MAJOR ${MAJOR_VERSION} )
  set ( LIBMPSSE_VERSION_MINOR ${MINOR_VERSION} )

  set ( LIBMPSSE_USE_FILE ${CMAKE_INSTALL_PREFIX}/${LIBMPSSE_CMAKE_CONFIG_DIR}/UseLibMPSSE.cmake )
  configure_file ( cmake/LibMPSSEConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/LibMPSSEConfig.cmake @ONLY )
  install ( FILES ${CMAKE_CURRENT_BINARY_DIR}/LibMPSSEConfig.cmake cmake/UseLibMPSSE.cmake
            DESTINATION ${LIBMPSSE_CMAKE_CONFIG_DIR}
          )
endif ()
