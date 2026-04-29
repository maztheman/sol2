cmake_minimum_required(VERSION 3.21)

project(sol2 LANGUAGES C CXX)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

if(PROJECT_IS_TOP_LEVEL)
    set(CMAKE_CXX_STANDARD 23)
    set(CMAKE_CXX_EXTENSIONS OFF)
    set(CPM_SOURCE_CACHE ${CMAKE_CURRENT_BINARY_DIR}/.cpm-cache)
endif()

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

include(CPM)

add_library(sol2 INTERFACE)
add_library(sol2::sol2 ALIAS sol2)

target_include_directories(sol2 INTERFACE 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
)

CPMAddPackage(
	NAME lua
	GIT_REPOSITORY https://github.com/lua/lua.git
	GIT_TAG v5.4.7
	VERSION 5.4.7
	DOWNLOAD_ONLY TRUE
)

if(lua_ADDED)
	# lua has no CMakeLists, so we create our own target
	file(GLOB lua_sources ${lua_SOURCE_DIR}/*.c)
	list(REMOVE_ITEM lua_sources "${lua_SOURCE_DIR}/lua.c" "${lua_SOURCE_DIR}/onelua.c")
	add_library(lua STATIC ${lua_sources})
	target_include_directories(lua BEFORE PUBLIC $<BUILD_INTERFACE:${lua_SOURCE_DIR}>)
	if(NOT WIN32)
		target_compile_definitions(lua PUBLIC LUA_USE_POSIX)
	endif()
	add_library(Lua::Lua ALIAS lua)
endif()

target_link_libraries(sol2 INTERFACE Lua::Lua)
target_compile_definitions(sol2 INTERFACE SOL_NO_LUA_HPP=1)
