
set(EXECUTABLES
    spiflash
    spiflashfast
    i2ceeprom
    ds1305
    gpio
    bitbang
)

# all executables are built from <executable name>.c
foreach(executable ${EXECUTABLES})
  add_executable(${executable} ${executable}.c)
  target_link_libraries(${executable} mpsse)
endforeach()

include_directories(${CMAKE_SOURCE_DIRECTORY}../)
