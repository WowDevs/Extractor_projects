# MaNGOS is a full featured server for World of Warcraft, supporting
# the following clients: 1.12.x, 2.4.3, 3.3.5a, 4.3.4a and 5.4.8
#
# Copyright (C) 2005-2016  MaNGOS project <http://getmangos.eu>
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA

set(EXECUTABLE_NAME "vmap-extractor")

# comment next line to disable debug mode
add_definitions("-DIOMAP_DEBUG")
add_definitions("-DNO_CORE_FUNCS")

include_directories(
    ${CMAKE_SOURCE_DIR}/src/shared
    ${CMAKE_SOURCE_DIR}/src/game/vmap
    ${CMAKE_SOURCE_DIR}/dep/g3dlite
    ${CMAKE_SOURCE_DIR}/dep/g3dlite/G3D
    ${CMAKE_SOURCE_DIR}/src/framework
    ${CMAKE_SOURCE_DIR}/dep/libmpq
    ${CMAKE_SOURCE_DIR}/dep/loadlib
    ${CMAKE_SOURCE_DIR}/src/tools/Extractor_projects/shared
    ${ACE_INCLUDE_DIR}
)

if(WIN32)
    # add resource file to windows build
    set(EXECUTABLE_SRCS ${EXECUTABLE_SRCS} vmap-extractor.rc)
endif()

add_executable(${EXECUTABLE_NAME} ${EXECUTABLE_SRCS}
    adtfile.cpp
    assembler.cpp
    model.cpp
    vmapexport.cpp
    wdtfile.cpp
    wmo.cpp
    ${CMAKE_SOURCE_DIR}/src/shared/Auth/md5.c
    ${CMAKE_SOURCE_DIR}/src/tools/Extractor_projects/shared/dbcfile.cpp
    ${CMAKE_SOURCE_DIR}/src/tools/Extractor_projects/shared/ExtractorCommon.cpp
)

if(NOT ACE_USE_EXTERNAL)
    target_link_libraries(${EXECUTABLE_NAME} ace)
else()
    target_link_libraries(${EXECUTABLE_NAME} ACE)
endif()

target_link_libraries(${EXECUTABLE_NAME} loadlib libmpq g3dlite vmap bzip2 zlib)

set(EXECUTABLE_LINK_FLAGS "")

if(UNIX)
    if(CMAKE_SYSTEM_NAME MATCHES "FreeBSD")
        set(EXECUTABLE_LINK_FLAGS "-Wl,--no-as-needed -lrt -pthread ${EXECUTABLE_LINK_FLAGS}")
    elseif(CMAKE_SYSTEM_NAME MATCHES "Linux")
        set(EXECUTABLE_LINK_FLAGS "-Wl,--no-as-needed -ldl -pthread -lrt ${EXECUTABLE_LINK_FLAGS}")
    endif()
endif()

if(APPLE)
    set(EXECUTABLE_LINK_FLAGS "-framework CoreServices ${EXECUTABLE_LINK_FLAGS}")
endif()

set_target_properties(${EXECUTABLE_NAME} PROPERTIES LINK_FLAGS "${EXECUTABLE_LINK_FLAGS}")


#Output the compiled exes to build/bin/$(Configuration)/tools directory on windows by default
if(WIN32)
    if ( MSVC )
        set_target_properties(${EXECUTABLE_NAME} PROPERTIES
            RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin/Release/tools
            RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/bin/Debug/tools
        )
    elseif ( MINGW )
        set_target_properties(${EXECUTABLE_NAME} PROPERTIES
            RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin/tools
            RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/bin/tools
        )
    endif()
endif()

install(TARGETS ${EXECUTABLE_NAME} DESTINATION "${BIN_DIR}/${TOOLS_DIR}")
if(WIN32 AND MSVC)
    install(FILES "${CMAKE_CURRENT_BINARY_DIR}/\${BUILD_TYPE}/${EXECUTABLE_NAME}.pdb" DESTINATION "${BIN_DIR}/${TOOLS_DIR}" CONFIGURATIONS Debug)
endif()
