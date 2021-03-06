CMAKE_MINIMUM_REQUIRED(VERSION 3.9)

PROJECT(mariadb_modern_cpp)

INCLUDE(cmake/all.cmake)

add_library(mariadb_modern_cpp INTERFACE)

target_include_directories(mariadb_modern_cpp INTERFACE 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/hdr>
    $<INSTALL_INTERFACE:include>
)

FIND_PATH(MARIADB_CONFIG_DIR mariadb_config PATHS /opt/bin /usr/bin /usr/local/bin)
IF(MARIADB_CONFIG_DIR)
  EXEC_PROGRAM(${MARIADB_CONFIG_DIR}/mariadb_config
    ARGS "--include"
    OUTPUT_VARIABLE MARIADB_INCLUDE_DIR)
  separate_arguments(MARIADB_INCLUDE_DIR)
  target_compile_options(mariadb_modern_cpp INTERFACE ${MARIADB_INCLUDE_DIR})
  EXEC_PROGRAM(${MARIADB_CONFIG_DIR}/mariadb_config
    ARGS "--libs_r"
    OUTPUT_VARIABLE MARIADB_LIBS)
  separate_arguments(MARIADB_LIBS)
  TARGET_LINK_LIBRARIES(mariadb_modern_cpp INTERFACE ${MARIADB_LIBS})
ELSE()
  message(FATAL_ERROR "no mariadb C library found")
ENDIF()

option(BUILD_TESTS "Build tests" OFF)
option(BUILD_FUZZING "Build fuzzing" OFF)

# test
if(BUILD_TESTS)
  add_subdirectory(test)
endif()

if(BUILD_FUZZING)
  add_subdirectory(fuzz_test)
endif()

# install lib
INSTALL(TARGETS mariadb_modern_cpp EXPORT ${PROJECT_NAME}Targets)

install(DIRECTORY
    ${CMAKE_CURRENT_SOURCE_DIR}/hdr/
  DESTINATION include
)


# install cmake config file
INSTALL(EXPORT ${PROJECT_NAME}Targets
  FILE ${PROJECT_NAME}Config.cmake
  DESTINATION lib/cmake/${PROJECT_NAME})
