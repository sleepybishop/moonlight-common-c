
cmake_minimum_required(VERSION 3.1)
project(moonlight-common-c LANGUAGES C)

SET(CMAKE_C_STANDARD 11)

find_package(OpenSSL 1.0.2 REQUIRED)

aux_source_directory(src SRC_LIST)
aux_source_directory(enet SRC_LIST)

set(NANORS_SRC_LIST
   "nanors/oblas_lite.c"
   "nanors/rs.c"
)
set(SRC_LIST
  ${SRC_LIST}
  ${NANORS_SRC_LIST}
)
set(VECTORIZATION_FLAGS "-O3 -march=native -funroll-loops")

if(MSVC)
  add_compile_options(/W4 /wd4100 /wd4232)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS=1 -D_CRT_NONSTDC_NO_DEPRECATE=1)
  link_libraries(ws2_32.lib qwave.lib winmm.lib)
else()
  add_compile_options(-Wall -Wextra -Wno-unused-parameter -Werror)
  set_property(SOURCE ${NANORS_SRC_LIST} PROPERTY COMPILE_FLAGS ${VECTORIZATION_FLAGS})
endif()

string(TOUPPER "x${CMAKE_BUILD_TYPE}" BUILD_TYPE)
if("${BUILD_TYPE}" STREQUAL "XDEBUG")
	add_definitions(-DLC_DEBUG)
else()
	add_definitions(-DNDEBUG)
endif()

add_definitions(-DHAS_SOCKLEN_T)

include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/enet/include
  ${CMAKE_CURRENT_SOURCE_DIR}/nanors
  ${OPENSSL_INCLUDE_DIR})

link_libraries(${OPENSSL_CRYPTO_LIBRARY})

add_library(moonlight-common-c SHARED ${SRC_LIST})
