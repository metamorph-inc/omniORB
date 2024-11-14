cmake_minimum_required(VERSION 3.15)

set(NAME omniorb)
set(VERSION 4.2.3)

project(${NAME} VERSION ${VERSION})

set(ORIGIN "\$\$ORIGIN")
set(BUILD "--build=${CMAKE_HOST_SYSTEM_PROCESSOR}-unknown-linux-gnu")

include(ExternalProject)

ExternalProject_Add(
    ${NAME}
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/omniORB
    BINARY_DIR build_omniORB
    CONFIGURE_COMMAND
    ${CMAKE_COMMAND} -E env LDFLAGS=-Wl,-rpath,'${ORIGIN}'
    ${CMAKE_SOURCE_DIR}/omniORB/configure --prefix=${CMAKE_INSTALL_PREFIX} ${BUILD}
    INSTALL_COMMAND make install
    COMMAND "sed" "-i.bak" "s=^#!.*python.*$=#!/usr/bin/env python="
            "${DESTDIR}${CMAKE_INSTALL_PREFIX}/bin/omniidl"
    COMMAND ${CMAKE_COMMAND} "-E" "rm" "${DESTDIR}${CMAKE_INSTALL_PREFIX}/bin/omniidl.bak"
)

ExternalProject_Add(
    ${NAME}py
    SOURCE_DIR ${CMAKE_SOURCE_DIR}/omniORBpy
    BINARY_DIR build_omniORBpy
    CONFIGURE_COMMAND
        ${CMAKE_COMMAND} -E env LDFLAGS=-Wl,-rpath,'${ORIGIN}:${ORIGIN}/../..' CPPFLAGS=-I${DESTDIR}${CMAKE_INSTALL_PREFIX}/include/omniORB4/internal
        ${CMAKE_SOURCE_DIR}/omniORBpy/configure --with-omniorb=${DESTDIR}${CMAKE_INSTALL_PREFIX} --prefix=${CMAKE_INSTALL_PREFIX} ${BUILD}
)

add_dependencies(${NAME}py ${NAME})

# dummy file for install target
install(FILES README.md DESTINATION share/${NAME}/)