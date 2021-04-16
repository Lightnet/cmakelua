cmake_minimum_required(VERSION 3.20) # FetchContent is available in 3.11+
#project name
project(lua)

option(LUA_BUILD_BINARY "Build lua binary" ON)
option(LUA_BUILD_COMPILER "Build luac compiler" ON)

#set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}) # /lib
#set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}) # /lib
#set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}) # /bin

#message("CMAKE_ARCHIVE_OUTPUT_DIRECTORY: ${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}") # workspace/build/Debug
#message("CMAKE_LIBRARY_OUTPUT_DIRECTORY: ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}") # workspace/build/Debug
#message("CMAKE_RUNTIME_OUTPUT_DIRECTORY: ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}") # workspace/build/Debug

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(BUILD_SHARED_LIBS ON)

# LUA ON
set(ENABLE_LUA ON)
set(USE_GIT_LUA OFF)
# missing luac file
set(LUA_GIT_VERSION v5.4.3) #not used
set(LUA_TARGET_VERSION LUA_COMPAT_5_4_3) #not used
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
        set(LUA_SRC_DIR ${lua_SOURCE_DIR})
      else()
        set(LUA_SRC_DIR ${lua_SOURCE_DIR}/src)
      endif()

      file(GLOB LUA_HEADER_FILES
        "${LUA_SRC}/*.h"
      )
      file(COPY ${LUA_HEADER_FILES} DESTINATION ${lua_BINARY_DIR}/include)
      set(${lua_INCLUDE_DIRS} ${lua_BINARY_DIR}/include)

    endif()
  endif()
endif()

#message("PROJECT_BINARY_DIR " ${PROJECT_BINARY_DIR})
link_directories (${PROJECT_BINARY_DIR})

# 1. lua static library
# how to rename library name?
# LIB
set(LUA_SRC
  ${LUA_SRC_DIR}/lapi.c 
  ${LUA_SRC_DIR}/lcode.c 
  ${LUA_SRC_DIR}/lctype.c 
  ${LUA_SRC_DIR}/ldebug.c 
  ${LUA_SRC_DIR}/ldo.c 
  ${LUA_SRC_DIR}/ldump.c 
  ${LUA_SRC_DIR}/lfunc.c 
  ${LUA_SRC_DIR}/lgc.c 
  ${LUA_SRC_DIR}/llex.c 
  ${LUA_SRC_DIR}/lmem.c 
  ${LUA_SRC_DIR}/lobject.c 
  ${LUA_SRC_DIR}/lopcodes.c 
  ${LUA_SRC_DIR}/lparser.c 
  ${LUA_SRC_DIR}/lstate.c 
  ${LUA_SRC_DIR}/lstring.c 
  ${LUA_SRC_DIR}/ltable.c 
  ${LUA_SRC_DIR}/ltm.c 
  ${LUA_SRC_DIR}/lundump.c 
  ${LUA_SRC_DIR}/lvm.c 
  ${LUA_SRC_DIR}/lzio.c 
  ${LUA_SRC_DIR}/lauxlib.c 
  ${LUA_SRC_DIR}/lbaselib.c 
  ${LUA_SRC_DIR}/lcorolib.c 
  ${LUA_SRC_DIR}/ldblib.c 
  ${LUA_SRC_DIR}/liolib.c 
  ${LUA_SRC_DIR}/lmathlib.c 
  ${LUA_SRC_DIR}/loslib.c 
  ${LUA_SRC_DIR}/lstrlib.c 
  ${LUA_SRC_DIR}/ltablib.c 
  ${LUA_SRC_DIR}/loadlib.c 
  ${LUA_SRC_DIR}/linit.c
  ${LUA_SRC_DIR}/lutf8lib.c
)
# LIB
add_library (lualib STATIC ${LUA_SRC})
target_compile_definitions(lualib PRIVATE ${LUA_TARGET_VERSION})
target_include_directories(lualib INTERFACE ${LUA_SRC_DIR})
set_target_properties(lualib PROPERTIES OUTPUT_NAME "lua") # rename the library output name

# DLL
add_library (luadll SHARED ${LUA_SRC})
target_compile_definitions(luadll PRIVATE ${LUA_TARGET_VERSION})
target_include_directories(luadll INTERFACE ${LUA_SRC_DIR})
set_target_properties(luadll PROPERTIES OUTPUT_NAME "lua") # rename the library output name

if(LUA_BUILD_BINARY)
  # 2. lua interpreter
  #link_directories(${PROJECT_BINARY_DIR})
  add_executable(lua ${LUA_SRC_DIR}/lua.c)
  target_link_libraries(lua lualib)
  if(UNIX)
    target_link_libraries( lua m )
  endif()
endif()

if(LUA_BUILD_COMPILER)
  # 3. lua compiler
  #link_directories(${PROJECT_BINARY_DIR})
  add_executable(luac ${LUA_SRC_DIR}/luac.c)
  target_link_libraries(luac lualib)
  if(UNIX)
    target_link_libraries( luac m )
  endif()
endif()