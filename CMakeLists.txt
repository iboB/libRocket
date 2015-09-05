#===================================
# Simplified CMakeLists for libRocket to be used in another project
# * Not concerned with executables and demos
# * Most configuration data expected from above
# * No Find* macros. Assuming cached CMake macros with paths to dependencies
# * No installation, framework bundles, pack or tests
# * Currently no Python support
#===================================

# We use the new OSX_ARCHITECTURES property
# and GNUInstallDirs module
cmake_minimum_required(VERSION 2.8.5)

if(COMMAND cmake_policy)
  cmake_policy(SET CMP0015 NEW)
endif(COMMAND cmake_policy)

# Enable the use of MACOSX_RPATH by default for CMake v3.0+; this effectively
# allows plug 'n' play functionality, so to speak -- the resulting shared
# library files can simply be copied over into the end-user's application
# bundle or framework bundle. No mucking around with install_name_tool.
#
# 	See also:
# cmake --help-policy cmp0042
# http://www.kitware.com/blog/home/post/510
if(POLICY CMP0042)
  cmake_policy(SET CMP0042 NEW)
endif(POLICY CMP0042)

project(libRocket C CXX)

set(LIBROCKET_VERSION_MAJOR 1)
set(LIBROCKET_VERSION_MINOR 3)
set(LIBROCKET_VERSION_PATCH 0)
set(LIBROCKET_VERSION_TWEAK 0)
set(PROJECT_VERSION ${LIBROCKET_VERSION_MAJOR}.${LIBROCKET_VERSION_MINOR}.${LIBROCKET_VERSION_PATCH}.${LIBROCKET_VERSION_TWEAK})

#===================================
# Plaform specific global hacks ====
#===================================

if(APPLE)
	# Disables naked builtins from AssertMacros.h which
	# This prevents naming collisions such as those from the check()
	# function macro with LuaType::check
	add_definitions(-D__ASSERT_MACROS_DEFINE_VERSIONS_WITHOUT_UNDERSCORES=0)
endif(APPLE)

#===================================
# Build options ====================
#===================================

if(NOT IOS)
	option(ROCKET_BUILD_SHARED_LIBS "Build shared libraries" ON)
endif(NOT IOS)

if(ROCKET_BUILD_SHARED_LIBS)
    set(BUILD_SHARED_LIBS ON)
endif()

set(BUILD_PYTHON_BINDINGS OFF) # currently disabled
option(BUILD_LUA_BINDINGS "Build Lua bindings" OFF)

if(APPLE)
	if(IOS)
		if(BUILD_SHARED_LIBS)
			message(FATAL_ERROR "BUILD_SHARED_LIBS must be OFF for iOS builds.  iOS does not support shared libraries.")
		endif(BUILD_SHARED_LIBS)
	endif(IOS)
endif(APPLE)

if(NOT BUILD_SHARED_LIBS)
	add_definitions(-DROCKET_STATIC_LIB)
endif()

#on windows, check for VC10 and fix the multiple compile target issue.
IF(WIN32)
  if(MSVC)
    if(${MSVC_VERSION} STREQUAL 1600 OR ${MSVC_VERSION} STRGREATER 1600)
      # message("Visual Studio 2010 (${MSVC_VERSION}) build fix at play (/FORCE:MULTIPLE)")
      # set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /FORCE:MULTIPLE")
    endif()
  endif()
ENDIF(WIN32)


#===================================
# Find dependencies ================
#===================================

# FreeType

# Boost and Python
#~ if(BUILD_PYTHON_BINDINGS)
#~ endif()

#Lua
if(BUILD_LUA_BINDINGS)
    # TODO
endif()


#===================================
# Setup paths ======================
#===================================

set(PROJECT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})

set(LIBROCKET_INCLUDE_DIR
    ${PROJECT_SOURCE_DIR}/Include
    CACHE STRING "libRocket include path"
)

include_directories(
    ${LIBROCKET_INCLUDE_DIR}
    ${FREETYPE_INCLUDE_DIR}
)

# Include list of source files
include(Build/cmake/FileList.cmake)

#===================================
# Build libraries ==================
#===================================

set(ALL_LIBS)

set(LIBRARIES Core Controls Debugger)

foreach(library ${LIBRARIES})
    set(NAME Rocket${library})
    list(APPEND ALL_LIBS ${NAME})

    add_library(${NAME}
                        ${${library}_HDR_FILES}
                        ${${library}_PUB_HDR_FILES}
                        ${MASTER_${library}_PUB_HDR_FILES}
			${${library}_SRC_FILES}
    )

    set_target_properties(${NAME} PROPERTIES
                           VERSION ${PROJECT_VERSION}
                           SOVERSION ${LIBROCKET_VERSION_MAJOR}
                           FOLDER ThirdParty/libRocket
    )
endforeach(library)

# Build python bindings
#~ if(BUILD_PYTHON_BINDINGS)
#~     set(LIBRARIES core controls)

#~     foreach(library ${LIBRARIES})
#~         set(NAME _rocket${library})
#~         list(APPEND ALL_LIBS ${NAME})

#~         add_library(${NAME} MODULE ${Py${library}_SRC_FILES}
#~                             ${Py${library}_HDR_FILES}
#~                             ${Py${library}_PUB_HDR_FILES}
#~         )

#~         set_target_properties(${NAME} PROPERTIES
#~             PREFIX ""
#~             FOLDER ThirdParty/libRocket
#~         )
#~     endforeach(library)
#~ endif()

# Build Lua bindings
if(BUILD_LUA_BINDINGS)
    set(LIBRARIES Core Controls)

    foreach(library ${LIBRARIES})
        set(NAME Rocket${library}Lua)
        list(APPEND ALL_LIBS ${NAME})

        add_library(${NAME} ${Lua${library}_SRC_FILES}
                            ${Lua${library}_HDR_FILES}
                            ${Lua${library}_PUB_HDR_FILES}
        )

        set_target_properties(${NAME} PROPERTIES
                           VERSION ${PROJECT_VERSION}
                           SOVERSION ${LIBROCKET_VERSION_MAJOR}
                           FOLDER ThirdParty/libRocket
        )
    endforeach(library)
endif()


#===================================
# Link libraries ===================
#===================================

link_directories(${FREETYPE_LINK_DIR})
target_link_libraries(RocketCore ${FREETYPE_LIB})
target_link_libraries(RocketControls RocketCore)
target_link_libraries(RocketDebugger RocketCore)

#~ if(BUILD_PYTHON_BINDINGS)
#~     target_link_libraries(_rocketcore RocketCore ${PY_BINDINGS_LINK_LIBS})
#~     target_link_libraries(_rocketcontrols RocketControls ${PY_BINDINGS_LINK_LIBS})
#~ endif()

if(BUILD_LUA_BINDINGS)
#~     target_link_libraries(RocketCoreLua RocketCore ${LUA_BINDINGS_LINK_LIBS})
#~     target_link_libraries(RocketControlsLua RocketControls RocketCoreLua ${LUA_BINDINGS_LINK_LIBS})
endif()

set(LIBROCKET_LINK_DIRS
    ${CMAKE_CURRENT_BINARY_DIR}
    CACHE STRING "libRocket link path"
)

set(LIBROCKET_LIBS ${ALL_LIBS}
    CACHE STRING "libRocket libraries"
)