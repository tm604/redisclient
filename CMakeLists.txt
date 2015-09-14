cmake_minimum_required(VERSION 2.6)

PROJECT(RedisClient)

set (VERSION_MAJOR 0)
set (VERSION_MINOR 4)
set (VERSION_PATCH 3)
set (PROJECT_VERSION_REVISION 1)
set (PROJECT_VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

FIND_PACKAGE(Boost COMPONENTS unit_test_framework system REQUIRED)

SET(CMAKE_CXX_FLAGS "-W -Wall -Wextra -Wshadow -fPIC -pthread")
SET(CMAKE_CXX_FLAGS_DEBUG "-O0 -g3 -DDEBUG ")
SET(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")

SET(SOURCES
    src/redisclient/impl/redisvalue.cpp
    src/redisclient/impl/redissyncclient.cpp
    src/redisclient/impl/redisparser.cpp
    src/redisclient/impl/redisclientimpl.cpp
    src/redisclient/impl/redisasyncclient.cpp
)

SET(HEADERS
    src/redisclient/config.h
    src/redisclient/version.h
    src/redisclient/redisvalue.h
    src/redisclient/redissyncclient.h
    src/redisclient/redisparser.h
    src/redisclient/redisclient.h
    src/redisclient/redisbuffer.h
    src/redisclient/redisasyncclient.h
    src/redisclient/impl/redisclientimpl.h
)

INCLUDE_DIRECTORIES(
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${Boost_INCLUDE_DIRS}
)

ADD_EXECUTABLE(parsertest tests/parsertest.cpp ${SOURCES} ${HEADERS})
ADD_LIBRARY(redisclient STATIC ${HEADERS} ${SOURCES})

TARGET_LINK_LIBRARIES(parsertest
    ${Boost_LIBRARIES}
)

TARGET_LINK_LIBRARIES(redisclient
    ${Boost_LIBRARIES}
)

INSTALL(DIRECTORY ${CMAKE_SOURCE_DIR}/src DESTINATION include)

ADD_CUSTOM_TARGET(test)
ADD_CUSTOM_COMMAND(TARGET test 
    POST_BUILD
    COMMAND ./parsertest
    COMMENT "Run tests"
    VERBATIM
)

ADD_SUBDIRECTORY(examples)

include (InstallRequiredSystemLibraries)
set (CPACK_PACKAGE_NAME "libredisclient")
set (CPACK_PACKAGE_VENDOR "")
set (CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
set (CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")
set (CPACK_PACKAGE_CONTACT "Tom Molesworth <tom@perlsite.co.uk>")
set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++ ASIO Redis client")
set (CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.debian")
set (CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set (CPACK_GENERATOR "TGZ;DEB")
# No need for this since we're header-only
set (CPACK_DEBIAN_PACKAGE_SHLIBDEPS "OFF")
string(TOLOWER "${CPACK_PACKAGE_NAME}" CPACK_PACKAGE_NAME_LOWERCASE)
find_program(DPKG_PROGRAM dpkg)
if(DPKG_PROGRAM)
	execute_process(
		COMMAND ${DPKG_PROGRAM} --print-architecture
		OUTPUT_VARIABLE CPACK_DEBIAN_PACKAGE_ARCHITECTURE
		OUTPUT_STRIP_TRAILING_WHITESPACE
		)
	set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}_${PROJECT_VERSION}-${PROJECT_VERSION_REVISION}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")   
else(DPKG_PROGRAM)
	set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}_${PROJECT_VERSION}-${PROJECT_VERSION_REVISION}_${CMAKE_SYSTEM_NAME}")
endif(DPKG_PROGRAM)
include (CPack)

