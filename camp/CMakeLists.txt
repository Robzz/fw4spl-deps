cmake_minimum_required(VERSION 3.0)

project(campBuilder)

include(ExternalProject)

include( ${CMAKE_SOURCE_DIR}/cmake/findBinpkgs/fw-boost.cmake )
set(CAMP_CMAKE_ARGS ${COMMON_CMAKE_ARGS})

if(ANDROID)
    if( ${ANDROID_STL} MATCHES "gnustl_shared" )
        set(CAMP_CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} --std=c++11")
        list(APPEND CAMP_CMAKE_ARGS
            -DCMAKE_CXX_FLAGS=${CAMP_CMAKE_CXX_FLAGS} )
    endif()
endif()

set(CAMP_VERSION 0.8.1)
set(CACHED_URL https://github.com/fw4spl-org/camp/archive/${CAMP_VERSION}.zip)

ExternalProject_Add(
    camp
    URL ${CACHED_URL}
    URL_HASH SHA256=e379fda7efeeaff6da592d4f5a769cb7af2396de3c646e629d86e755ca541b74
    DOWNLOAD_DIR ${ARCHIVE_DIR}
    DOWNLOAD_NAME camp-${CAMP_VERSION}.zip
    DEPENDS boost
    INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
    CMAKE_ARGS ${CAMP_CMAKE_ARGS}
)

ExternalProject_Add_Step(camp CopyConfigFileToInstall
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/cmake/findBinpkgs/FindCAMP.cmake ${CMAKE_INSTALL_PREFIX}/FindCAMP.cmake
    COMMENT "Install configuration file"
    DEPENDEES install
)
