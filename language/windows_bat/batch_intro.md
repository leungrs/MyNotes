# Windows批处理程序



批处理程序里面包含的是windows的命令，一般有两个后缀：.bat, .cmd。

其中cmd文件中允许的命令比bat文件多，cmd只支持windows 2000及以上的操作系统。而bat没有这个限制。



批处理程序目前没有好的调试器，只能通过echo在程序里打印日志来定位问题。



### 批处理文件中的特殊符号

- ^ 是转义符

- ；分号可以把一个命令的多个目标连接起来，如：

  ```dir c:\;d:\;e:\```相当于执行了这条dir命令执行了3次。

- ，逗号相当于空格，比如：```dir,c:\```



### 目录中包含空格的处理

1. 使用双引号括起来，如：cd "c:\Program Files" ，  cd "Local Settings"
2. 使用忽略空格后的前6个字符加上~1，如：cd c:\Progra~1,  cd LocalS~1

### 基本的程序构造

首先声明：批处理程序中，不区分大小写。

#### 定义变量

新的变量不需要定义，直接使用，如果没有赋值，值为空字符串（""），通常自己定义的变量名最好用小写，不要覆盖了常见的系统环境变量，比如：TEMP，PATH, USERPROFILE等。

#### 变量赋值

使用set给变量赋值，如set a=123，等号前后不能有空格。

使用/A开关时，就可以在变量赋值时使用算术运算，如：set /A  foo=2+2 ，结果foo等于4

#### 获取变量的值

使用两个百分号把变量括起来，表示读取变量的值。可以读取：

- 自己定义的变量，如：%foo%
- 系统提供的静态变量，如：%PATH%
- DOS提供的动态变量，如：%DATE%, %random%, %cd%，注意这些变量都不区分大小写。

#### 变量的作用域

默认情况下，变量的作用域是在整个命令行会话。使用setlocal命令，可以让变量的作用域只在定义它的脚本中。执行了endlocal, exit, 或者脚本执行完毕了，那么变量就不存在了。

一个真实的例子就是在脚本文件中重新定义自己的PATH变量。

#### 一些特殊的变量

1. 通过命令行传递给脚本的变量，读取他们的时候，使用一个百分号+参数序号，如：%1, %2
2. 使用感叹号代替百分号，把变量括起来，表示变量延迟解析，也就是在用到的时候才解析。如!var!。

#### 脚本中处理命令行参数

1. 使用%0-9获取命令行参数，其中0表示脚本文件本身，1-9依次表示第几个参数，没有的话，参数的值为空字符串。这种方式只能获取脚本的前9个参数，多余的参数，要借助于shift命令来获取。

2. 命令行参数还支持一些有用的参数替换，可以用来解析文件路径，时间戳，文件大小等，常用的有：

   - %~I 表示去掉第I个参数的双引号，如：%~0 ，表示去掉第0个参数，也就是脚本文件名中的双引号。

   - %fI 表示把第I个参数解析成全路径。

   - %fsI，表示把第I个参数解析成全路径时，使用DOS8.3的短路径形式。c:\progra~1就是c:\Program Files的DOC8.3短路径形式。

   - %dpI，表示把第I个参数解析成他的父路径的全路径。

   - %nxI，表示把第I个参数解析成只包含文件名和扩展名的形式，比如在脚本中可以这样写：

     ```shell
     echo %nx0: some message  -- 这样的话，就知道message是从哪个脚本中来的。
     ```

#### 编写脚本的头部的最佳实践

给脚本最开头，加上如下代码，如：

```shell
SETLOCAL ENABLEEXTENSIONS
set me=%~n0
set parent=%~dp0
```

第一句使得脚本中定义的变量不会污染命令行的整个会话，同时开启了命令扩展。

第二，三句分布包含脚本的文件名和路径到一个变量，方便后面的使用。

#### 命令或者脚本的返回值

对返回值的一般约定是：正常返回0，不常住返回非0，大于0或者小于0都可以。

环境变量ERRORLEVEL保持了最最后的一个脚本或者命令的返回值。不过有一些命令的执行不会更改这个值，

比如：echo, if, set等。

使用如下方式可以检查返回值：

1. 使用不等于操作符: NEQ（not equal to)

   ```shell
   IF %ERRORLEVEL% NEQ 0 (
   REM 不等于0，出错了，打印出错信息
   )
   ```

2. 在IF语句中，支持ERRORLEVEL直接加数字，此时ERRORLEVEL不需要%， 表示的含义就是ERRORLEVEL大于或者等于后面的数字。

   ```shell
   IF ERRORLEVEL 1 (
     REM 返回值大于等于1，出错了，打印出错信息
   )
   ```

有一种超酷的方法来根据前一个命令的返回值，来确定是否执行接下来的命令，可以省略IF：

1. 在前一个命令成功后，才执行第二个命令的话，使用&&，如：

   ```shell
   SomeCommand.exe && echo SomeCommand.exe succeeded!
   ```

2. 在前一个命令失败后，才执行第二个命令的话，使用||，如：

   ```shell
   # 失败后，打印一条消息及返回码
   SomeCommand.exe || echo SomeCommand.exe failed with return code %ERRORLEVEL%
   # 或者采用如下方式，使用带/B的开关退出，当前脚本的执行上下文，而不是退出command命令行提示符进程。
   SomeCommand.exe || exit /B 1
   # 或者借助脚本中隐含的一个位置标签:EOF，跳到脚本的最后，也就是退出脚本的执行。
   SomeCommand.exe || goto :EOF
   ```

   

   



### DOS命令介绍

#### echo命令

echo message   -- 显示message

echo on|off   -- 开启|关闭命令回显功能

#### @命令

表示不显示后面接的命令，经常见到的用法是：

@echo off  --  关闭回显，但是执行过程中，不会显示出这条命令

#### cd命令

chdir的简写形式，显示或者改变当前目录。语法为：

```shell
CD|CHDIR [/D] [drive:][path]
```

常见的用法有：

1. cd /d  d:\software -- 带上了/d选项，表示切换目录的时候，如果当前磁盘不是d的话，也会切换到d
2. cd  d:\software  -- 没有带上/d选项，如果当前磁盘不是d盘的话，不会真的改变当前目录。但是当你切换到d盘后，该目录就会变成当前目录。
3. cd \  表示切换到当前盘的跟目录。
4. cd  什么参数也不带，就显示当前目录。
5. cd D:   指带一个磁盘驱动参数，就是显示指定驱动的当前目录。

#### rem命令

注释，一般和@命令结合使用，如：

@rem 这是一条注释，当然如果首先关闭了回显，那也没有必要加上@了。

有些高级玩家，用两个冒号(::)来表示注释，因为冒号是给代码位置打标签的，两个冒号会被忽略。

#### goto命令

转到一个标签处继续执行，如：

```shell
goto noparams
@rem check params if null
:noparams
echo Usage: xxx
goto end

:end
echo prog exited!!!
```

#### pause命令

暂停程序执行，输出如下提示信息：

Press any key to continue...

#### call命令

从当前批处理程序中调用另一个批处理程序。

#### choice命令

提供选项给用户选择，语法为：

CHOICE [/C choices] [/N] [/CS] [/T timeout /D choice] [/M text]

该命令返回用户所选择的项的索引，从1开始。返回的索引值赋值给了环境变量ERRORLEVEL。在批处理文件中要按索引值的降序依次来检测他的值。典型的用法如下：

```shell
choice /C YNC /T 100 /D y /M "Yes，No或者Cancel, 100秒后自动选择由/D指定的默认值Yes" 
if errorlevel 3 goto label3
if errorlevel 2 goto label2
if errorlevel 1 goto lable1

:label3
xxxxx
goto end

:label2
xxx
goto end

:label1
xxxx
goto end

:end
exit
```

#### if命令

有三种格式：

1. if "参数" == "字符串"  待执行的命令

2. if exist 文件名   待执行的命令

3. if errorlevel  数字 goto label   满足条件时：表示errorlevel大于等于数字

   if not errorlevel 数字  goto label

#### for命令

有如下几种常见用法

1. FOR %variable IN (set)  DO command [command-parameters]

   遍历特定的集合，其中的set是一个或者多个名称，从空格，逗号或者tab分割。

   如果set中的某一项包含通配符，表示匹配当前文件夹下的文件名。

   ```shell
   for %i in (aaa, bbb, ccc) do echo %i   -- 输出三行，分别是aaa,bbb,ccc
   for %i in (*.txt) do echo %i  -- 输出当前目录下所有的txt文件名
   ```

2. FOR /D %variable IN (set) DO command [command-parameters]

   用法和1类似，只不过如果set中包含通配符，通配符只和目录匹配，不和文件名匹配。

   个人理解：/D的D代表Directory目录

3. FOR /R [[drive:]path] %variable IN (set) DO command [command-parameters]

   在指定的目录中递归地执行for命令，如果没有指定目录，就是在当前目录下递归

   个人理解：/R的R代表Recursive递归

4. FOR /L %variable IN (start, step, end)  DO command [command-parameters]

   产出一个序列，遍历这个序列中的每一项。序列从start开始，步长step，到end结束，

   包含start和end。

   个人理解：/L的L代表List列表

5. FOR /F ["options"] %variable IN (file-set)  DO command [command-parameters]

   FOR /F ["options"] %variable IN ("string")  DO command [command-parameters]

   FOR /F ["options"] %variable IN ('command')  DO command [command-parameters]

   这三种用法，是用来解析文件内容，字符串，命令输出内容的。

   个人理解：/F 估计代表File文件，表示用来解析文件。其中：

   双引号引起来的option用来指定解析方式，有如下一些选项：

   - eol=c                       #表示文件中的注释以c字符开始，只能指定单个字符，解析时会忽略。

   - skip=n                     # 指定文件开始解析时，忽略前n行

   - delims=xxx             # 指定分隔符集合，用来替换默认的分隔符集（空格，tab）

   - tokens=x,y,m-n      # 表示分割后，给哪些token分配变量。

   - usebackq                # 是否使用反引号(`)来表示一个命令，不指定这个选项的话：

     ​                                  单引号表示命令，双引号表示普通字符串，不带引号的表示文件

     ​                                  反之，如果指定了这个选项的话：

     ​                                   反引号表示命令，双引号的表示文件，单引号的表示字符串。

   举例如下：

   ```shell
   FOR /F "eol=; tokens=2,3* delims=, " %i in (myfile.txt) do @echo %i %j %k
   上面的命令含义为：
   解析myfile.txt每一行，忽略分号开头的行。用逗号或者空格分割，分割后第二个token和第三个token分配给变量i, j。*变量表示剩余的token，都分配给k变量。
   ```


###  命令的串联执行

命令有三种串联方式：

1. &  每个命令都按顺序执行一遍，不管前面的命令是否执行成功。
2. &&  短路执行，如果前面一个执行失败了，后面的就不会执行了。
3. ||  短路执行，如果前面一个执行成功了，后面的就不会执行了。
4. |  命令管道，前一个命令的输出，作为后一个命令的输入。



### 输入输出重定向



​                                          