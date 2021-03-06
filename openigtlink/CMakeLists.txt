cmake_minimum_required(VERSION 3.0)

project(openigtlinkBuilder)

include(ExternalProject)

if(UNIX)
    # Allows to reuse server sockets after they are closed.
    # We need that otherwise we can't restart a sender on the same port during a application run.
    # The define flag looks strange, but yes this code seems to come from VTK
    # See Source/igtlSocket.cxx:116
    set(OPENIGTLINK_CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DVTK_HAVE_SO_REUSEADDR")
else()
    set(OPENIGTLINK_CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
endif()

set(OPENIGTLINK_CMAKE_ARGS ${COMMON_CMAKE_ARGS}
                           -DBUILD_TESTING:BOOL=OFF
                           -DBUILD_EXAMPLES:BOOL=OFF
                           -DBUILD_DOCUMENTATION:BOOL=OFF
                           -DBUILD_SHARED_LIBS:BOOL=ON
                           -DOpenIGTLink_PROTOCOL_VERSION_2:BOOL=ON
                           -DCMAKE_CXX_FLAGS:STRING=${OPENIGTLINK_CMAKE_CXX_FLAGS}
)

set(CACHED_URL https://github.com/openigtlink/OpenIGTLink/archive/5a501817c2da52e81db4db3eca6dd5111f94fed9.tar.gz)


if(ANDROID)
    # Android doesn't need to link with pThread lib so we need to remove it from target link library
    # Furthermore lib OpenIGTLinkUtil needs to be linked with libm
    set(PATCH_CMD ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/patch/Source/CMakeLists.txt "<SOURCE_DIR>/Source/CMakeLists.txt"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different ${CMAKE_CURRENT_SOURCE_DIR}/patch/Source/igtlutil/CMakeLists.txt "<SOURCE_DIR>/Source/igtlutil/CMakeLists.txt")
endif()

if(APPLE)
    list(APPEND PATCH_CMD
        COMMAND ${PATCH_EXECUTABLE} -p1 -i ${CMAKE_CURRENT_SOURCE_DIR}/patch/macos_socket_deadlock_workaround.diff -d <SOURCE_DIR>
    )
endif()

ExternalProject_Add(
    openigtlink
    URL ${CACHED_URL}
    URL_HASH SHA256=dae54b032e061dd3f3ddd37291fdfc7fdb15181c5544ad8d74a8b93f06bc697f
    DOWNLOAD_DIR ${ARCHIVE_DIR}
    DOWNLOAD_NAME openigtlink.tar.gz
    INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
    PATCH_COMMAND ${PATCH_CMD}
    CMAKE_ARGS ${OPENIGTLINK_CMAKE_ARGS}
)

ExternalProject_Add_Step(openigtlink CopyConfigFileToInstall
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/../cmake/findBinpkgs/fw-IGTL.cmake ${CMAKE_INSTALL_PREFIX}/fw-IGTL.cmake
    COMMENT "Install configuration file"
    DEPENDEES install
)
