# ~~~
# ####################################################################
# Description:  CMakeLists.txt
#
#               This file, 'CMakeLists.txt', implements build system
#               rules for AM335x PRU PACKAGE project
#
# Copyright (C) 2021    Jakub Fišer  <jakub DOT fiser AT eryaf DOT com>
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
#
# Redistributions in binary form must reproduce the above copyright
# notice, this list of conditions and the following disclaimer in the
# documentation and/or other materials provided with the
# distribution.
#
# Neither the name of Texas Instruments Incorporated nor the names of
# its contributors may be used to endorse or promote products derived
# from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
#
# #####################################################################
# ~~~

add_executable(pasm)

set(PASM_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasm.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasmpp.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasmexp.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasmop.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasmdot.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasmstruct.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasmmacro.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/path_utils.c)

set(PASM_PRIVATE_HEADER_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasm.h
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pasmdbg.h
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/path_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/pru_ins.h)

set(PASM_MANUAL_FILES ${CMAKE_CURRENT_SOURCE_DIR}/pasm.1)

target_sources(pasm PRIVATE ${PASM_SOURCES} ${PASM_PRIVATE_HEADER_FILES})

target_compile_options(pasm PRIVATE "-Wall")

target_compile_definitions(pasm PRIVATE "_UNIX_")

install(TARGETS pasm RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
                             COMPONENT PRU_Assembler_Binary)

install(
  FILES ${PASM_MANUAL_FILES}
  DESTINATION "${CMAKE_INSTALL_MANDIR}/man1"
  COMPONENT PRU_Assembler_Manual)

add_executable(pasm_test_path_utils_O3)
add_executable(pasm_test_path_utils_g)

set(PASM_TEST_PATH_UTILS_SOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/test/path_utils_test.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/path_utils.c)

set(PASM_TEST_PATH_UTILS_PRIVATE_HEADER_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/path_utils.h)

target_sources(
  pasm_test_path_utils_O3 PRIVATE ${PASM_TEST_PATH_UTILS_SOURCES}
                                  ${PASM_TEST_PATH_UTILS_PRIVATE_HEADER_FILES})
target_sources(
  pasm_test_path_utils_g PRIVATE ${PASM_TEST_PATH_UTILS_SOURCES}
                                 ${PASM_TEST_PATH_UTILS_PRIVATE_HEADER_FILES})

target_compile_options(pasm_test_path_utils_O3 PRIVATE "-Wall" "-O3")
target_compile_options(pasm_test_path_utils_g PRIVATE "-Wall" "-g")

target_compile_definitions(pasm_test_path_utils_O3 PRIVATE "_UNIX_")
target_compile_definitions(pasm_test_path_utils_g PRIVATE "_UNIX_")

set_target_properties(
  pasm_test_path_utils_O3
  PROPERTIES RUNTIME_OUTPUT_DIRECTORY
             "${CMAKE_CURRENT_BINARY_DIR}/tests/pasm_test_path_utils_O3")
set_target_properties(
  pasm_test_path_utils_g
  PROPERTIES RUNTIME_OUTPUT_DIRECTORY
             "${CMAKE_CURRENT_BINARY_DIR}/tests/pasm_test_path_utils_g")

add_custom_command(
  TARGET pasm_test_path_utils_O3
  POST_BUILD
  COMMAND
    "${CMAKE_COMMAND}" "-E" "copy_if_different"
    "${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/test/path_utils_test.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tests/pasm_test_path_utils_O3/path_utils_test.c"
)
add_custom_command(
  TARGET pasm_test_path_utils_g
  POST_BUILD
  COMMAND
    "${CMAKE_COMMAND}" "-E" "copy_if_different"
    "${CMAKE_CURRENT_SOURCE_DIR}/pasm_source/test/path_utils_test.c"
    "${CMAKE_CURRENT_BINARY_DIR}/tests/pasm_test_path_utils_g/path_utils_test.c"
)

add_test(
  NAME testPASMPathUtils-O3
  COMMAND pasm_test_path_utils_O3
  WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/tests/pasm_test_path_utils_O3")
add_test(
  NAME testPASMPathUtils-g
  COMMAND pasm_test_path_utils_g
  WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/tests/pasm_test_path_utils_g")
