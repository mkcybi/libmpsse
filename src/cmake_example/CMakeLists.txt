cmake_minimum_required ( VERSION 2.8 )

project ( example C )

find_package ( LibMPSSE NO_MODULE REQUIRED )
include ( ${LIBMPSSE_USE_FILE} )

add_executable ( example main.c )
target_link_libraries( example ${LIBMPSSE_LIBRARIES} )

install ( TARGETS example
          DESTINATION bin )
          
