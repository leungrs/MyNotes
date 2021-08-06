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



### npm如何的配置？

npm执行命令时，按从高到低的优先级从如下6个地方获取配置：

1. 命令行，如--proxy http://server:port，就将proxy的值设为http://server:port

2. 环境变量，以npm_config_开头的环境变量，如npm_config_proxy=http://server:port

3. 用户的配置文件，通过npm config get userconfig命令可以查看用户配置文件路径。如我本机上执行这个命令的输出结果如下：

   ```
   C:\Users\Administrator>npm config get userconfig
   C:\Users\Administrator\.npmrc
   ```

   使用npm config set <key> <value> 设置的配置为用户级别的配置。

4. 全局的配置文件，通过npm config get globalconfig可以获取全局配置文件路径

   ```
   C:\Users\Administrator>npm config get globalconfig
   C:\Users\Administrator\AppData\Roaming\npm\etc\npmrc
   ```

   使用npm config set <key> <value>  --global设置的配置为全局级别的配置。

5. 内置的配置文件，也就是npm安装目录下的npmrc文件

6. 默认配置。npm本身有默认的配置参数，如果以上5条都没有对应的配置，就使用默认的。使用```npm config list -l```查看当前所有配置及默认配置。



### npm的目录结构

npm会把包安装到你机器上的不同位置，简单来说就是：

1. 本地安装，就会在把包安装在当前路径下的node_modules文件夹下。
2. 全局-g安装，就会把包安装到prefix对应路径下的node_modules文件夹下，不同的操作系统prefix默认值不同。也可以通过npm config set prefix=xxxx -g改变。
3. 如果你需要require它，那么就使用本地安装。
4. 如果你需要在命令行里执行它，那么就全局安装。
5. 如果两者都需要，那么就本地和全局都安装一下。

#### node_modules文件夹

npm安装包到node_modules文件夹下，如果是本地安装你可以通过require(packagename)来加载包的主模块，也就是package.json中main指定的模块。也可以通过require(packagename/lib/子模块路径)来加载包的其他子模块。scope包会有一个@scopename的文件夹，这个scope名字的包都在这个文件夹下。



可执行文件全局安装到prefix下（windows）或者prefix/bin下（unix），本地安装到node_modules/.bin下，在scripts脚本中可以直接使用本地安装的可执行文件（本地可执行文件安装目录会添加到PATH环境变量）。



缓存文件夹默认unix上为~/.npm， windows上为%AppData%\npm-cache。可以通过npm  cache控制。



本地安装时，npm查找node_modules的逻辑：

1. 从当前路径开始，查看路径中是否包含package.json或者node_modules文件夹。如果存在，那么就把这个路径当做真实的当前路径，安装包到此目录下的node_modules文件夹下，不存在就新建。
2. 如果不存在就继续按照相同路径查找上级目录，直到根目录。
3. 如果还是没有找到的话，就在当前路径下新建node_modules文件夹进行安装。

安装时，包先下载到缓存文件夹中，然后unpack到node_modules文件夹下。



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



### package.json文件是什么？

这个文件包含很多配置内容，其中的许多内容又受到config配置的影响。常用的配置如下：

1. name  和 version，包名和版本，如果需要publish的话，是必须的，否则也不是必须的。

2. files，可选。用来指定哪些文件发布的时候是需要包含的，默认是所有的都包含。

3. main，可选，默认为index.js。导出的主模块，别人require你的包时，导入的就是这个模块。

4. browser，如果你的包只是为了在浏览器中使用，比如引用了window对象，那么就使用browser而不使用main

5. bin，许多包都有一个或者多个可执行文件需要安装到PATH中，这个配置用来指向命令名和对应的js文件。

6. scripts，是个字典，key为生命周期事件，值为执行的脚本，比如npm test 执行的脚本就在这里指定。

   不指定scripts的话，scripts有如下的默认值：

   ```javascript
   "scripts": {
       "start": "node server.js",
       "install": "node-gyp rebuild"
   }
   ```

7. dependencies，是一个字典，描述包的正式依赖。

8. devDependencies，和7类似，描述只在开发阶段用到的依赖。

9. peerDependencies，表示包可以和某个库或者宿主工具兼容，但并不强依赖它，必须可以作为它的一个插件。

10. optionalDependencies， 可选的依赖。

11. bundledDependencies，指定在npm pack时，需要一起打包进gzip文件中的package。

12. private，如果设置为false，那么就不能publish

13. workspaces，可选。是一个文件模式的数组，用来设置一系列本地文件系统中的路径。

14. directories: 指定package的lib, bin, docs，man等目录。



### package locks

使用package-lock.json或者npm-shrinkwrap.json来锁定依赖包的版本，这样方便在新的机器或者新的环境进行移植。其中npm-shrinkwrap.json的优先级高于package.json。类似于python中的pip frozen。



npm-shrinkwrap.json是通过npm shrinkwrap命令生成的。

### package.json中的scripts配置

在scripts中用户可以指定任意的脚本，通过key-value的形式，key表示命令的名字，value是执行命令时真实的执行脚本。通过npm run key来执行对应的命令。



处理用户自定义的脚本外，npm内置了一些命令的名字，npm在执行具体的任务时，如果发现scripts中定义了某个内置的命令名，那么它对应的脚本就会自动执行。这些一般叫做生命周期事件。这些内置的命令名有：

- prepare：在执行npm pack打包一个tarball前、在本地执行一个不带参数的npm install前、在执行npm publish发布一个包前会执行这个命令指定的脚本。
- prepublish：和prepare的生命周期一样，这个命令已经过期，不推荐使用了。如果有prepare，它在prepare之前执行。
- prepublishonly: 在包prepare和pack前，只对publish命令生效。
- prepack, 在打包出一个tarball前，如命令：npm pack, npm publish。
- postpack, 在打包出一个tarball，并输出到目的位置后执行。



一般使用场景有：

- 使用prepublish，在发布前完成coffeescript的编译，js的压缩，依赖资源的获取等等。



script执行时，npm的一些配置信息和当前的进程信息是可以获取到的：

- path：如果你的package的一个依赖module里有可执行程序，那么可以直接在scripts中使用，因为它们所在的路径在执行script的时候加到path环境变量中。
- package.json中的变量：package.json中的属性，会设置到环境变量中区，通过process.env.npm_package_xxx获取。比如xxx可以是name, version等等。
- npm的config参数，也会作为环境变量，通过process.env.npm_config_xxx获取。比如xxx可以是root, prefix, registry等等。
- npm_lifecycle_event环境变量被设置成当前正在执行的命令名，比如：install, test, prepare等等。



钩子脚本：

通过在node_modules/.hooks/{eventname}文件夹下放一个可执行文件，就给某个生命周期事件关联上了一个全局的钩子。



最佳实践：

- 不返回非0值作为错误码然后退出，应该让脚本继续执行，有错误打印相关信息。
- npm本身能干的事情，不要定义另外的scripts。
- 检查环境变量中的值来确定在什么地方放置内容，比如检查npm_config_rootbin来获取bin的目录，而不是直接指定死一个位置/usr/local/bin。
- 不要给脚本加上sudo
- 不要使用install，使用.gyp来指定编译过程。

### node的模块是什么？

node的模块就是在node_modules文件夹下的，可以通过nodejs的require方法导入的一个文件或者文件夹：

1. 如果是文件，那必须是JavaScript文件。
2. 如果是文件夹，文件夹中必须包含package.json文件，并且文件中必须包含main属性。

由于模块不一定需要一个package.json（比如单个js文件），所以不是所有的node模块都是包，只有文件夹形式的模块才是一个package。

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



#### npm install

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



#### npm config

1. 用户级配置：npm config set key value
2. 全局配置：npm config set key value --global



#### npm run

是npm run-script的简写。

可以在scripts中定义一些生命周期事件对应的脚本，一个命令的前置和后置命令，使用pre和post，如：

```json
// 使用npm run compress时, 会在执行compress的前后分别执行pre和post对应的命令
{
    "scripts": {
        "precompress": "压缩前要执行的命令",
        "compress": "正式执行压缩的命令",
        "postcompress": "压缩后执行的命令"
    }
}
```





### 一个应用拆成多个模块





### 使用Scoped包和团队成员共享私有代码

#### scope简介

npm从版本2以后，引入了scope的概念，目的是解决包名冲突的问题。给包加上一个scope，那么不同的scope就可以使用相同的包名，彼此不产生冲突。scope的名字一般使用个人或者组织的名字命名。在@和/字符中间的所有字符串就是这个包的scope。比如：```@npm/package-name```，它的scope名就是npm。



### npm配置代理和镜像

配置代理：

```shell
npm config set proxy http://username:password@server:port
npm config set https-proxy https://username:password@server:port
```

配置镜像:

```shell
# 淘宝镜像
npm config set registry https://registry.npm.taobao.org/
```

