
PROJECT(MICA)
CMAKE_MINIMUM_REQUIRED(VERSION 3.21)


set(CMAKE_BUILD_TYPE Release CACHE STRING "Build configuration (Release/Debug)")
set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release")
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()


OPTION(DEBUG "Enable debugging" OFF)
OPTION(LTO "Use link time optimization" OFF)

# OPTION(AVX2 "Enable AVX2" ON)



set(CMAKE_C_STANDARD 23)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR}/src)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)


set(proj_compile_options
    -Wall -Wextra -Wzero-as-null-pointer-constant
    -Wsign-conversion -Wconversion
    -Wno-unused-function -march=native
    -pthread -g
)
if (CMAKE_C_COMPILER_ID STREQUAL "Clang")
    set(proj_compile_options ${proj_compile_options}
        -Wno-unknown-warning-option
        -Wno-expansion-to-defined
        -Wno-unused-local-typedef
    )
endif()


IF(LTO)

  MESSAGE(STATUS "Link time optimization is enabled (to disable, run `cmake .. -DLTO=OFF`)")
  set(proj_compile_options ${proj_compile_options} -flto)
  SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -flto")

ELSE(LTO)

  MESSAGE(WARNING "Link time optimization is DISABLED (to enable, run `cmake .. -DLTO=ON`). Performance will be low.")

ENDIF(LTO)
add_compile_options(${proj_compile_options})


# IF(AVX2)

#   MESSAGE(STATUS "AVX2 is enabled (to disable, run `cmake .. -DAVX2=OFF`)")
#   ADD_DEFINITIONS(-mavx2)
#   SET(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -mavx2")

# ELSE(LTO)

#   MESSAGE(WARNING "AVX2 is DISABLED (to enable, run `cmake .. -DAVX2=ON`). Performance will be low.")

# ENDIF(AVX2)



SET(SOURCES ${SOURCES} src/mica/alloc/hugetlbfs_shm.cc)
SET(SOURCES ${SOURCES} src/mica/transaction/timestamp.cc)
SET(SOURCES ${SOURCES} src/mica/util/config.cc)
SET(SOURCES ${SOURCES} src/mica/util/cityhash/city_mod.cc)
SET(SOURCES ${SOURCES} src/mica/util/siphash/siphash24.c)
SET_SOURCE_FILES_PROPERTIES(src/mica/util/siphash/siphash24.c PROPERTIES LANGUAGE CXX)
SET(SOURCES ${SOURCES} src/mica/util/lcore.cc)
SET(SOURCES ${SOURCES} src/mica/util/stopwatch.cc)
SET(SOURCES ${SOURCES} src/mica/util/zipf.cc)

SET(LIBRARIES ${LIBRARIES} rt numa pthread)



IF(LTO)

  ADD_EXECUTABLE(test_atomics src/mica/test/test_atomics.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_atomics ${LIBRARIES})

  ADD_EXECUTABLE(test_tsc_sync src/mica/test/test_tsc_sync.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tsc_sync ${LIBRARIES})

  ADD_EXECUTABLE(test_tx src/mica/test/test_tx.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tx ${LIBRARIES})

  ADD_EXECUTABLE(test_tx_index src/mica/test/test_tx_index.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tx_index ${LIBRARIES})

ELSE(LTO)

  ADD_LIBRARY(common ${SOURCES})

  ADD_EXECUTABLE(test_atomics src/mica/test/test_atomics.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_atomics ${LIBRARIES})

  ADD_EXECUTABLE(test_tsc_sync src/mica/test/test_tsc_sync.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tsc_sync ${LIBRARIES})

  ADD_EXECUTABLE(test_tx src/mica/test/test_tx.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tx ${LIBRARIES})

  ADD_EXECUTABLE(test_tx_index src/mica/test/test_tx_index.cc ${SOURCES})
  TARGET_LINK_LIBRARIES(test_tx_index ${LIBRARIES})

ENDIF(LTO)
