set(C_SOURCES)
auto_sources(files "*.c" "RECURSE" "${CMAKE_CURRENT_SOURCE_DIR}/mcrouter")
foreach (file ${files})
  if (${file} MATCHES "/(test|examples)/")
    list(REMOVE_ITEM files ${file})
  endif()
  if (${file} MATCHES "/mc_fbtrace_info.c$")
    list(REMOVE_ITEM files ${file})
  endif()
endforeach()
# ascii_client.c is generated from lib/mc/ascii_client.rl using ragel
list(APPEND C_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/ascii_client.c")
list(APPEND C_SOURCES ${files})

set(CXX_SOURCES)
auto_sources(files "*.cpp" "RECURSE" "${CMAKE_CURRENT_SOURCE_DIR}/mcrouter")
foreach (file ${files})
  if (${file} MATCHES "(/(test|examples)/|Test.cpp$)")
    list(REMOVE_ITEM files ${file})
  endif()
endforeach()
# McAsciiParser-gen.cpp is generated from mcrouter/lib/network/McAsciiParser.rl
# using ragel
# So ran the command found in mcrouter/Makefile.am manually and put the file
# here -- this avoids linking errors
list(APPEND CXX_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/McAsciiParser-gen.cpp")
list(APPEND CXX_SOURCES ${files})

add_definitions(-DNO_LIB_GFLAGS)
add_definitions(-DLIBMC_FBTRACE_DISABLE)
add_definitions(-DHAVE_CONFIG_H)

find_package(Boost 1.51.0 COMPONENTS context regex system filesystem REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

include_directories("${HPHP_HOME}/third-party")
include_directories("${HPHP_HOME}/third-party/mcrouter/")
#include_directories("${HPHP_HOME}/third-party/mcrouter/mcrouter/lib/fbi")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-variable")
add_library(mcrouter STATIC ${C_SOURCES} ${CXX_SOURCES})
target_link_libraries(mcrouter folly ${Boost_LIBRARIES})
