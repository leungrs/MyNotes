# npm教程



### 什么是npm？

npm也就是node package manager的缩写，npm包含指下面三个组件：

1. npm的网站：https://npmjs.com/，可以用来查找包。
2. npm的命令行工具，node自带了这个工具，用来安装js软件包，比如：npm install cnpm。
3. js软件包的注册中心: registry.npmjs.com，包含大量js软件包及其元数据信息的公共的数据库。



> node自带的npm，可能版本不是最新的，使用如下方式升级npm：
>
> windows: ```npm install npm -g```
>
> linux: ```sudo npm install npm@latest -g```



### node的包是什么？

node的package可以是如下的任何一种形式：

1. 一个文件夹，里面包含程序代码，通过一个package.json来描述。
2. 是1中所说的文件夹的gzip格式压缩文件。
3. 是一个url，url指向2中所说的gzip文件。
4. 是 ```<name>@<version>```的形式，发布在一个包的注册中心。
5. 是```<name>@<tag>``` 的形式，发布在一个包的注册中心。
6. 是```<name>```的形式，发布在一个注册中心，其实是把5中的tag固定为latest了。
7. 是一个git的远程地址，可以解析为1中所说的文件夹格式。



那么用来描述一个包的package.json有包含什么内容呢？



### package.json文件

这个文件包含很多配置内容，其中的许多内容又受到config配置的影响。常用的配置如下：

1. name  和 version，包名和版本，如果需要publish的话，是必须的，否则也不是必须的。
2. files，可选。用来执行哪些文件发布的时候是需要包含的。默认是所有的都包含。
3. main，可选，默认为index.js。导出的主模块，别人require你的包时，导入的就是这个模块。
4. browser，如果你的包只是为了在浏览器中使用，比如引用了window对象，那么就使用browser而不使用main
5. bin，许多包都有一个或者多个可执行文件需要安装到PATH中，这个配置用来指向命令名和对应的js文件。
6. scripts，是个字典，key为生命周期事件，值为执行的脚本，比如npm test 执行的脚本就在这里指定。
7. dependencies，是一个字典，描述包的正式依赖。
8. devDependencies，和7类似，描述只在开发阶段用到的依赖。
9. peerDependencies，表示包可以和某个库或者宿主工具兼容，但并不强依赖它，必须可以作为它的一个插件。
10. optionalDependencies， 可选的依赖。
11. bundledDependencies，指定在npm pack是，需要一起打包进gzip文件中的package。
12. private，如果设置为false，那么就不能publish
13. workspaces，可选。是一个文件模式的数组，用来设置一系列本地文件系统中的路径。



### npm命令行工具简介

npm是nodejs平台的javascript包管理器，它可以管理js模块，把它们放到正确的文件目录下，方便node找到它们，同时npm还能智能的管理js模块的冲突。



npm也是高度可配置的，用来支持广泛的使用场景。大部分的应用场景，是使用它来查找、安装、开发和发布node应用程序。使用方式如下：

```sh
npm <command> [args]
```

通过```npm help```可以获取可用的command列表。常用的command有：
install, init, start, search, version, run, publish, test, list, config, uninstall, unpublish，
view, whoami, create, get, repo, rebuild, set, prefix, stop等等。



npm给常用的一些command起了别名，也就是简写形式，一个命令可以有好几个别名，用来减少命令行的输入，常用的别名及其原名有：c/config/，ls/la/ll/list, i/in/ins/inst/insta/instal/isnt/isnta/isntal/add/install, r/rm/remove/un/unlink/uninstall，up/update/upgrade等。

> 注意：默认情况下，npm是从npm公司的公共注册中心找包的：https://registry.npmjs.org

你可以配置npm使用其他兼容的包注册中心，你甚至可以建立自己的包注册中心。比如你就可以配置使用国内淘宝的npm包注册中心。



### npm常用命令



#### 包的安装：npm install

这个命令用来安装一个包，以及它依赖的包。如果这个包中含有如下三个文件，那么就按如下的优先级从这些文件中确定包的依赖包：

* npm-shrinkwrap.json
* package-lock.json
* yarn.lock

否则就从package.json中获取依赖。



常用的形式如下：

1. npm install jquery  安装jquery最新版本
2. npm install jquery@<tag>  安装jquery的某个tag版本
3. npm install jquery@3.6.0 安装jquery的某个具体版本
4. npm install jquery@">=3.1.0 <=3.6.0" 安装某个版本范围内的一个版本。



#### 配置: npm config

使用```npm config list -l```查看当前所有配置及默认配置。







### 一个应用拆成多个模块





### 使用Scoped模块和团队成员共享私有代码