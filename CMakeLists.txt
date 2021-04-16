cmake_minimum_required(VERSION 3.20) # FetchContent is available in 3.11+
#project name
project(lua)

option(LUA_BUILD_BINARY "Build lua binary" OFF)
option(LUA_BUILD_COMPILER "Build luac compiler" OFF)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}) # /lib
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}) # /lib
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}) # /bin

message("CMAKE_ARCHIVE_OUTPUT_DIRECTORY: ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}") # workspace/build/Debug
message("CMAKE_LIBRARY_OUTPUT_DIRECTORY: ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}") # workspace/build/Debug
message("CMAKE_RUNTIME_OUTPUT_DIRECTORY: ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}") # workspace/build/Debug

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(BUILD_SHARED_LIBS ON)

# LUA ON
set(ENABLE_LUA ON)
set(USE_GIT_LUA OFF)
# missing luac file
set(LUA_GIT_VERSION v5.4.3) #not used
set(LUA_GITHUB https://github.com/lua/lua.git) # not

set(LUA_URL https://www.lua.org/ftp/lua-5.4.3.tar.gz) # used

set(LUA_SRC "")
set(LUA_HEADER_FILES "")

if(WIN32)
  add_definitions( -D_CRT_SECURE_NO_WARNINGS )
endif()

if(ENABLE_LUA)
  # Set this to the minimal version you want to support
  find_package(lua QUIET) # Requires at least version 2.5.0
  # You could change the QUIET above to REQUIRED and remove this if() clause
  # This part downloads raylib and builds it if it's not installed on your system
  if (NOT lua_FOUND) # If there's none, fetch and build raylib
    include(FetchContent)
    if(USE_GIT_LUA)
      message("GIT HUB INIT...")
      FetchContent_Declare(
        lua
        GIT_REPOSITORY ${LUA_GITHUB}
        GIT_TAG ${LUA_GIT_VERSION}
      )
    else()
      message("URL INIT...")
      FetchContent_Declare(
        lua
        URL ${LUA_URL}
      )
    endif()

    FetchContent_GetProperties(lua)
    if (NOT lua_POPULATED) # Have we downloaded raylib yet?
      set(FETCHCONTENT_QUIET NO)
      FetchContent_Populate(lua)

      if(USE_GIT_LUA)
        set(LUA_SRC ${lua_SOURCE_DIR})
      else()
        set(LUA_SRC ${lua_SOURCE_DIR}/src)
      endif()

      file(GLOB LUA_HEADER_FILES
        "${LUA_SRC}/*.h"
      )
      file(COPY ${LUA_HEADER_FILES} DESTINATION ${lua_BINARY_DIR}/include)
      set(${lua_INCLUDE_DIRS})

    endif()
  endif()
endif()

#message("PROJECT_BINARY_DIR " ${PROJECT_BINARY_DIR})
link_directories (${PROJECT_BINARY_DIR})

# 1. lua static library
# how to rename library name?
# LIB
add_library (lualib 
  STATIC 
  ${LUA_SRC}/lapi.c 
  #${LUA_SRC}/lauxlib.c #? https://gist.github.com/baiyanhuang/4054106
  ${LUA_SRC}/lcode.c 
  ${LUA_SRC}/lctype.c 
  ${LUA_SRC}/ldebug.c 
  ${LUA_SRC}/ldo.c 
  ${LUA_SRC}/ldump.c 
  ${LUA_SRC}/lfunc.c 
  ${LUA_SRC}/lgc.c 
  ${LUA_SRC}/llex.c 
  ${LUA_SRC}/lmem.c 
  ${LUA_SRC}/lobject.c 
  ${LUA_SRC}/lopcodes.c 
  ${LUA_SRC}/lparser.c 
  ${LUA_SRC}/lstate.c 
  ${LUA_SRC}/lstring.c 
  ${LUA_SRC}/ltable.c 
  ${LUA_SRC}/ltm.c 
  ${LUA_SRC}/lundump.c 
  ${LUA_SRC}/lvm.c 
  ${LUA_SRC}/lzio.c 
  ${LUA_SRC}/lauxlib.c 
  ${LUA_SRC}/lbaselib.c 
  ${LUA_SRC}/lcorolib.c 
  ${LUA_SRC}/ldblib.c 
  ${LUA_SRC}/liolib.c 
  ${LUA_SRC}/lmathlib.c 
  ${LUA_SRC}/loslib.c 
  ${LUA_SRC}/lstrlib.c 
  ${LUA_SRC}/ltablib.c 
  ${LUA_SRC}/loadlib.c 
  ${LUA_SRC}/linit.c
  ${LUA_SRC}/lutf8lib.c
)
set_target_properties(lualib PROPERTIES OUTPUT_NAME "lua") # rename the library output name
# DLL
add_library (luadll 
  SHARED  
  ${LUA_SRC}/lapi.c 
  #${LUA_SRC}/lauxlib.c #? https://gist.github.com/baiyanhuang/4054106
  ${LUA_SRC}/lcode.c 
  ${LUA_SRC}/lctype.c 
  ${LUA_SRC}/ldebug.c 
  ${LUA_SRC}/ldo.c 
  ${LUA_SRC}/ldump.c 
  ${LUA_SRC}/lfunc.c 
  ${LUA_SRC}/lgc.c 
  ${LUA_SRC}/llex.c 
  ${LUA_SRC}/lmem.c 
  ${LUA_SRC}/lobject.c 
  ${LUA_SRC}/lopcodes.c 
  ${LUA_SRC}/lparser.c 
  ${LUA_SRC}/lstate.c 
  ${LUA_SRC}/lstring.c 
  ${LUA_SRC}/ltable.c 
  ${LUA_SRC}/ltm.c 
  ${LUA_SRC}/lundump.c 
  ${LUA_SRC}/lvm.c 
  ${LUA_SRC}/lzio.c 
  ${LUA_SRC}/lauxlib.c 
  ${LUA_SRC}/lbaselib.c 
  ${LUA_SRC}/lcorolib.c 
  ${LUA_SRC}/ldblib.c 
  ${LUA_SRC}/liolib.c 
  ${LUA_SRC}/lmathlib.c 
  ${LUA_SRC}/loslib.c 
  ${LUA_SRC}/lstrlib.c 
  ${LUA_SRC}/ltablib.c 
  ${LUA_SRC}/loadlib.c 
  ${LUA_SRC}/linit.c
  ${LUA_SRC}/lutf8lib.c
)
set_target_properties(luadll PROPERTIES OUTPUT_NAME "lua") # rename the library output name

if(LUA_BUILD_BINARY)
  # 2. lua interpreter
  #link_directories(${PROJECT_BINARY_DIR})
  add_executable(lua ${LUA_SRC}/lua.c)
  target_link_libraries(lua lualib)
  if(UNIX)
    target_link_libraries( lua m )
  endif()
endif()

if(LUA_BUILD_COMPILER)
  # 3. lua compiler
  #link_directories(${PROJECT_BINARY_DIR})
  add_executable(luac ${LUA_SRC}/luac.c)
  target_link_libraries(luac lualib)
  if(UNIX)
    target_link_libraries( luac m )
  endif()
endif()