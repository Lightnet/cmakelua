# cmakelua

# LICENSE: MIT

# Packages:
 * Lua 5.4.3
 * cmake 3.20.1

# Information:
  Testing setup build to run lua executes, lib and dll files. CMake will build the files base on compile tool and get the src file download by using FetchContent.

  By Using the cmake command builds.

# fetch files:
  There are two which can be downloaded. Github lua and the other is their main website lua.org.

  * https://www.lua.org/download.html
  * https://github.com/lua/lua

# Build:

```bat
:: Create a build directory
mkdir build 
:: change build directory and get CMakeList.txt config.
cd build && cmake ..
:: Build the project by default tool compiler to current hardware
cmake --build . 
```

```
build/Debug
 * lua.exe
 * lua.lib
 * lua.dll
 * luac.exe
 * etc...
```

# Links:
 * https://github.com/walterschell/Lua
 * https://gist.github.com/baiyanhuang/4054106
 * https://github.com/lua/lua
 * https://www.lua.org/
 * https://cmake.org/cmake/help/latest/module/FindLua.html

# Notes:
  * Lua 5.4.3 github missing luac.c file.
  * VS2019 Window 10 64 Bit.