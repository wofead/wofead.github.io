# Lua的编译和安装

## 方法

推荐在 `C:\` 下面建立一个 `local` 文件夹，用于像 linux 下 /usr/local 或者 /opt 一样来安装自定义的工具和库。

1. 下载下载 [MinGW Distro](https://nuwen.net/mingw.html)，这是 MinGW 的一个便捷版，可以免去平常安装 MinGW 了再从 mirror 上拉取软件包的麻烦操作。

2. 双击 mingw，路径填写 `C:\local`，然后点击 Extract 等待解压完成

3. 将 `C:\local\MinGW\bin` 加入到环境变量 `Path` 中。

4. 打开命令提示符，输入 `where gcc`，查看是否配置成功。

5. 下载 Lua 源码，官方地址 http://www.lua.org/ftp/ 

6. 将下载的 `lua-5.3.5.tar.gz` 解压放到 `C:\local`下。

7. 新建一个文本文件，改名为 `build.bat`，里面放入以下脚本内容（主要功能是创建需要的文件夹，编译lua，将成品拷贝到相应文件夹中并设置一些环境变量）：

   ```bash
   @echo off
   :: ========================
   :: build.bat
   ::
   :: build lua to dist folder
   :: tested with lua-5.3.5
   :: based on:
   :: https://medium.com/@CassiusBenard/lua-basics-windows-7-installation-and-running-lua-files-from-the-command-line-e8196e988d71
   :: ========================
   setlocal
   :: you may change the following variable’s value
   :: to suit the downloaded version
   set work_dir=%~dp0
   :: Removes trailing backslash
   :: to enhance readability in the following steps
   set work_dir=%work_dir:~0,-1%
   set lua_install_dir=%work_dir%\dist
   set compiler_bin_dir=%work_dir%\tdm-gcc\bin
   set lua_build_dir=%work_dir%
   set path=%compiler_bin_dir%;%path%
   cd /D %lua_build_dir%
   make PLAT=mingw
   echo.
   echo **** COMPILATION TERMINATED ****
   echo.
   echo **** BUILDING BINARY DISTRIBUTION ****
   echo.
   :: create a clean “binary” installation
   mkdir %lua_install_dir%
   mkdir %lua_install_dir%\doc
   mkdir %lua_install_dir%\bin
   mkdir %lua_install_dir%\include
   mkdir %lua_install_dir%\lib
   copy %lua_build_dir%\doc\*.* %lua_install_dir%\doc\*.*
   copy %lua_build_dir%\src\*.exe %lua_install_dir%\bin\*.*
   copy %lua_build_dir%\src\*.dll %lua_install_dir%\bin\*.*
   copy %lua_build_dir%\src\luaconf.h %lua_install_dir%\include\*.*
   copy %lua_build_dir%\src\lua.h %lua_install_dir%\include\*.*
   copy %lua_build_dir%\src\lualib.h %lua_install_dir%\include\*.*
   copy %lua_build_dir%\src\lauxlib.h %lua_install_dir%\include\*.*
   copy %lua_build_dir%\src\lua.hpp %lua_install_dir%\include\*.*
   copy %lua_build_dir%\src\liblua.a %lua_install_dir%\lib\liblua.a
   echo.
   echo **** BINARY DISTRIBUTION BUILT ****
   echo.
   %lua_install_dir%\bin\lua.exe -e "print [[Hello!]];print[[Simple Lua test successful!!!]]"
   echo.
   
   :: configure environment variable
   :: https://stackoverflow.com/a/21606502/4394850
   :: http://lua-users.org/wiki/LuaRocksConfig
   :: SETX - Set an environment variable permanently.
   :: /m Set the variable in the system environment HKLM.
   setx LUA "%lua_install_dir%\bin\lua.exe" /m
   setx LUA_BINDIR "%lua_install_dir%\bin" /m
   setx LUA_INCDIR "%lua_install_dir%\include" /m
   setx LUA_LIBDIR "%lua_install_dir%\lib" /m
   
   pause
   ```

8. 右键 `build.bat` 以管理员身份运行。成功时如图所示，会把 lua 编译到自动新建的 `dist` 文件夹下，并自动设置 `LUA`, `LUA_BINDIR`, `LUA_INCDIR`, 'LUA_LIBDIR` 这四个环境变量（因为设置这几个环境变量所以需要管理员权限。

9. 将 `C:\local\lua-5.3.5\dist\bin` 加入到环境变量 `Path` 中。

10. 再次打开命令提示符，输入 `where lua`，可以看到路径。输入 `lua` 可以启动 lua REPL。至此全部完成。