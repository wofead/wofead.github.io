# 记一次Bat脚本的学习和实现

使用bat脚本实现文件的拷贝和执行其它命令。进行svn的更新和提交。

[toc]

## 文件的拷贝

文件的拷贝分为两种情况，一是拷贝单个文件，而是拷贝多个文件。

首先我们考虑拷贝一个文件,使用copy函数，前面第一个参数是要拷贝的文件名字，后面的是要复制的目录已经新的名字，其中新的名字可以不要，这样就是和之前的名字一样了。

```bash
@echo off ::打开执行之后的操作提示和结果
set curPath=%cd% ::当前的目录
copy %curPath%\test.txt %cd%\testCopy.txt
pause
```

使用xcopy来拷贝目录下的所有文件及文件夹。一般还有一些参数需要携带。

1. /y:相同文件直接覆盖
2. /i:没有文件夹,则创建
3. /e:表示要复制空目录
4. /s:复制非空的目录和子目录。

```bash
@echo off
set curPath=%cd%

xcopy %curPath% %cd%\next /y ::/y相同文件直接覆盖，还有其他的参数：i:没有文件夹
pause
```

## 相对路径

我们可以使用`cd`命令来切换当前的目录，用来执行一些需要当前目录的操作。

我们可以通过`%cd%`和`%~dp0`来获取相对路径，他们两个还是存在区别的，一个是当前命令行所在的目录，一个是当前脚本所在的目录。

怎对于`%~dp0`的解释：`%0`为当前批处理文件，`%~d0 `是指批处理所在的盘符，`%~dp0` 是指批处理所在的盘符加路径

然后通过`../和./`来切换目录。

## 常用的特殊变量

1. `%PATH%  `:环境变量的字符串
2. ` %DATE%`:日期
3. `%CD%`:当前目录

## ERRORLEVEL

如果执行不成功进行错误处理。

```bash
svn update %curPath% %toPath% %templatePath%
IF ERRORLEVEL 0 echo success ::代表执行成功
IF ERRORLEVEL 1 echo you don't reg svn envoriment ::代表执行不成功
```

还有另一种使用方式：

```bash
svn update %curPath% %toPath% %templatePath%
IF %ERRORLEVEL% == 0 echo success ::代表执行成功
IF %ERRORLEVEL% == 1 echo you don't reg svn envoriment ::代表执行不成功
```

## SVN

我们使用svn一般分为update和commit，在update的时候可以更新多个目录也可以更新一个目录。在使用命令的前提是，配置好svn的环境变量。

```bash
svn update path1 path2 path3
```

接下来就是commit相关的了，在这里我们需要注意一点：直接使用`svn commit -m "annotation" path`它只会提交更改的，删除的和新增的都不会提交，这个和我们操作svn图形界面是一样的。所以我们需要特殊处理一下。

我们需要使用`svn st`命令来获取当前目录下文件的状态，"A"则执行添加动作，"!"则执行删除动作。

```bash
for /f "eol=M tokens=1,2 delims= " %%i in ('svn st') do (
    IF "%%i" == "?" (
        svn add "%%j"
    ) ELSE IF "%%i" == "A" (
        svn add "%%j"
    ) ELSE IF "%%i" == "!" (
        svn del "%%j"
    )
)
@echo off
set curPath=%cd%
set dPath=%cd%\next\
set ddPath=%cd%\next\next
for  %%i in (%curPath% %dPath% %ddPath%) do echo %%i
pause
```

在这里我们需要详细的解释一些for函数。

* for、in和do是for语句的关键字，它们三个缺一不可
* %I是for语句中对形式变量的引用，即使变量l在do后的语句中没有参与语句的执行，也是必须出现的
* in之后，do之前的括号不能省略
* in 后面的括号里面的内容可以是字符串，变量以及命令，如果是**命令需要用单引号引起来**。字符串则双引号。
* 括号里可以填多个数据，使用空格都好或等号分开，默认他们是多个数据
* /f:将命令行的值赋值给变量
* 使用 `delims=` 指定分隔符，/f 默认是以空格或者Table 键作为分隔符
* `tokens=1,2` 表示当前行按`delims`分割后的第1，2列值，定义 %%a 则第2列为%%b（*表示全部列）
* `eol=M` 表示忽略带M的行

```bash
@echo off
for %%i in (*.*) do echo "%%i" ::所有文件
pause
for %%i in (*.txt) do echo "%%i" ::.txt文件
pause

```

## 接收用户的输入

`set /p annotation=`然后annotation变量里面存放的就是用户输入的内容。

## code

最后附上完整的代码：

```bash
@echo off

::当前盘符
set curPath=%cd%
cd ../../client/Tool/ExcelToLua/excel
set toPath=%cd%
xcopy %curPath% %toPath% /y
cd ../
call %cd%/ExcelToLuaBombMan_Debug.exe

::需要更新和提交的目录 当前目录 curPath toPath templatePath
cd ../../Develop/Game/Src/Common/Config/Template
set templatePath=%cd%
::set defaultSvnPath = "C:\Program Files\TortoiseSVN\bin"
::set PATH=%PATH%;%defaultSvnPath%
svn update %curPath% %toPath% %templatePath%
IF ERRORLEVEL 1 echo you don't reg svn envoriment
echo =================
echo Input this commit annotation:
set /p annotation=
echo =================
for  %%i in (%curPath% %toPath% %templatePath%) do (
    cd %%i
    for /f "eol=M tokens=1,2 delims= " %%j in ('svn st') do (
        IF "%%j" == "?" (
            svn add "%%k"
        ) ELSE IF "%%j" == "A" (
            svn add "%%k"
        ) ELSE IF "%%j" == "!" (
            svn del "%%k"
        ) 
    )
)
svn commit -m %annotation% %curPath% %toPath% %templatePath%

pause

```

