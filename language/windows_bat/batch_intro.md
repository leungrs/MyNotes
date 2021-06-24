# Windows批处理文件



批处理文件里面包含的是windows的命令，一般有两个后缀：.bat, .cmd。

其中cmd文件中允许的命令比bat文件多，cmd只支持windows 2000及以上的操作系统。而bat没有这个限制。



### 命令介绍

#### echo命令

echo message   -- 显示message

echo on|off   -- 开启|关闭命令回显功能

#### @命令

表示不显示后面接的命令，经常见到的用法是：

@echo off  --  关闭回显，但是执行过程中，不会显示出这条命令

#### rem命令

注释，一般和@命令结合使用，如：

@rem 这是一条注释，当然如果首先关闭了回显，那也没有必要加上@了。

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

该命令返回用户所选择的项的索引，从1开始。返回的索引值赋值给了环境变量ERRORLEVEL。在批处理文件中要按索引值的降序依次来检测他的值。

```shell
choice /C YNC /T 100 /D y /M "Yes，No或者Cancel, 100秒后自动选择由/D指定的默认值Yes" 
```

#### if命令

有三种格式：

1. if "参数" == "字符串"  待执行的命令

2. if exist 文件名   待执行的命令

3. if errorlevel  数字  待执行的命令

   if not errorlevel 数字  待执行的命令

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

   - eol=c                       #表示文件中的注释以c字符开始，只能指定单个字符

   - skip=n                     # 指定文件开始解析时，忽略前n行

   - delims=xxx             # 指定分隔符集合，用来替换默认的分隔符集（空格，逗号，tab）

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

   

   ​                                          