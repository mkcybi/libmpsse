cmake_minimum_required ( VERSION 2.8 )

find_package ( LibFTDI1 NO_MODULE REQUIRED )
include ( ${LIBFTDI_USE_FILE} )

add_definitions(-DPACKAGE_VERSION="1.3")

set(SOURCE_FILES mpsse.c fast.c support.c)

# static mpsse library
add_library(mpsse-static STATIC
	${SOURCE_FILES}
)
target_link_libraries(mpsse-static ${LIBFTDI_LIBRARIES} )

# shared mpsse library
add_library(mpsse SHARED
	${SOURCE_FILES}
)
target_link_libraries(mpsse ${LIBFTDI_LIBRARIES} )

install(TARGETS mpsse
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
)

set_target_properties(mpsse-static PROPERTIES OUTPUT_NAME "mpsse")
install(TARGETS mpsse-static
	ARCHIVE DESTINATION lib
)

set(HEADER_FILES mpsse.h)

install(FILES ${HEADER_FILES}
	DESTINATION include/libmpsse
	COMPONENT headers
)


add_subdirectory(examples)


# swig and python related things
option ( PYTHON_BINDINGS "Build python bindings via swig" ON )
option ( LINK_PYTHON_LIBRARY "Link against python libraries" ON )

if ( PYTHON_BINDINGS )
  find_package ( SWIG )
  find_package ( PythonLibs )
  find_package ( PythonInterp )
endif ()

if ( SWIG_FOUND AND PYTHONLIBS_FOUND AND PYTHONINTERP_FOUND )
  include ( UseSWIG )
  include_directories ( BEFORE ${CMAKE_CURRENT_SOURCE_DIR} )
  include_directories ( ${PYTHON_INCLUDE_DIRS} )
  link_directories ( ${CMAKE_CURRENT_BINARY_DIR}/../src )

  swig_add_module ( mpsse python mpsse.i )

  if ( LINK_PYTHON_LIBRARY )
    swig_link_libraries ( mpsse ${PYTHON_LIBRARIES} )
  endif ()

  execute_process ( COMMAND ${PYTHON_EXECUTABLE} -c "from distutils import sysconfig; print( sysconfig.get_python_lib( plat_specific=True, prefix='${CMAKE_INSTALL_PREFIX}' ) )"
                    OUTPUT_VARIABLE _ABS_PYTHON_MODULE_PATH
                    OUTPUT_STRIP_TRAILING_WHITESPACE )

  get_filename_component ( _ABS_PYTHON_MODULE_PATH ${_ABS_PYTHON_MODULE_PATH} ABSOLUTE )
  file ( RELATIVE_PATH _REL_PYTHON_MODULE_PATH ${CMAKE_INSTALL_PREFIX} ${_ABS_PYTHON_MODULE_PATH} )

  set ( PYTHON_MODULE_PATH
        ${_REL_PYTHON_MODULE_PATH}
      )

  INSTALL ( FILES ${CMAKE_CURRENT_BINARY_DIR}/_mpsse.so DESTINATION ${PYTHON_MODULE_PATH} )
  INSTALL ( FILES ${CMAKE_CURRENT_SOURCE_DIR}/mpsse.py DESTINATION ${PYTHON_MODULE_PATH} )
endif ()
