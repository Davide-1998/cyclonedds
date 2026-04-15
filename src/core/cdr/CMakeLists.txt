#
# Copyright(c) 2022 ZettaScale Technology and others
#
# This program and the accompanying materials are made available under the
# terms of the Eclipse Public License v. 2.0 which is available at
# http://www.eclipse.org/legal/epl-2.0, or the Eclipse Distribution License
# v. 1.0 which is available at
# http://www.eclipse.org/org/documents/edl-v10.php.
#
# SPDX-License-Identifier: EPL-2.0 OR BSD-3-Clause
#


set(CYCLONEDDS_PROJECT_NAME ${PROJECT_NAME}) # Save to avoid override with CDR project
cmake_minimum_required(VERSION 3.16)
project(cdr VERSION 11.0.1 LANGUAGES C)


set(srcs_cdr
  "${CMAKE_CURRENT_LIST_DIR}/src/dds_cdrstream.c"
  "${CMAKE_CURRENT_LIST_DIR}/src/dds_cdrstream_keys.part.h"
  "${CMAKE_CURRENT_LIST_DIR}/src/dds_cdrstream_write.part.h")

set(hdrs_private_cdr
  "${CMAKE_CURRENT_LIST_DIR}/include/dds/cdr/dds_cdrstream.h")

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/../../../cmake/Modules")
include(GenerateExportHeader)

# Add ddsrt sources because cdr by not linking directly to ddsrt cannot know some symbols
# definitions
# TODO # Link directly DDSRT
set(CYCLONEDDS_DDSRT_DIR "${CMAKE_CURRENT_LIST_DIR}/../../ddsrt")
set(srcs_ddsrt
  "${CYCLONEDDS_DDSRT_DIR}/src/bswap.c"
  "${CYCLONEDDS_DDSRT_DIR}/src/heap/posix/heap.c"
  "${CYCLONEDDS_DDSRT_DIR}/src/hopscotch.c"
  "${CYCLONEDDS_DDSRT_DIR}/src/log.c"
  "${CYCLONEDDS_DDSRT_DIR}/src/mh3.c"
  "${CYCLONEDDS_DDSRT_DIR}/src/string.c"
)

# We are not checking neither for FREERTOS or ZEPHYR, DDSRT should the one delegated for the work. Link to it
if(WIN32)
  list(APPEND srcs_ddsrt
    "${CYCLONEDDS_DDSRT_DIR}/src/sync/windows/sync.c"
    "${CYCLONEDDS_DDSRT_DIR}/src/time/windows/time.c"
    "${CYCLONEDDS_DDSRT_DIR}/src/threads/windows/threads.c"
  )
else()
  list(APPEND srcs_ddsrt
    "${CYCLONEDDS_DDSRT_DIR}/src/sync/posix/sync.c"
    "${CYCLONEDDS_DDSRT_DIR}/src/time/posix/time.c"
    "${CYCLONEDDS_DDSRT_DIR}/src/threads/posix/threads.c"
  )
endif()

add_library(cdr ${srcs_cdr} ${hdrs_private_cdr} ${srcs_ddsrt})
add_library(${CYCLONEDDS_PROJECT_NAME}::cdr ALIAS cdr)

target_include_directories(cdr PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>"
                                      "$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>"
                                      "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/../../ddsrt/include>"
                                      "$<BUILD_INTERFACE:${CYCLONEDDS_DDSRT_DIR}/src>" # Needed for threads_priv.h
                                      "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/../ddsc/include>")

configure_file(${CMAKE_CURRENT_LIST_DIR}/../../ddsrt/include/dds/config.h.in include/dds/config.h)
configure_file(${CMAKE_CURRENT_LIST_DIR}/../../ddsrt/include/dds/features.h.in include/dds/features.h)
configure_file(${CMAKE_CURRENT_LIST_DIR}/../../ddsrt/include/dds/version.h.in include/dds/version.h)

generate_export_header(
cdr BASE_NAME DDS EXPORT_FILE_NAME include/dds/export.h)

install(
  TARGETS cdr
  EXPORT "${CYCLONEDDS_PROJECT_NAME}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}" COMPONENT lib
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT lib
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}" COMPONENT lib
)

install(
  DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/include/"
            "${CMAKE_CURRENT_BINARY_DIR}/include/"
            "${CMAKE_CURRENT_LIST_DIR}/../../ddsrt/include/"
            "${CMAKE_CURRENT_LIST_DIR}/../ddsc/include/"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
  COMPONENT dev)
