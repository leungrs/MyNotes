# vue笔记



### 单文件组件（single file component）

在一个文件中（通常以.vue为扩展名），使用类似于html的语法描述的vue组件，称为单文件组件。这个文件中主要包含template，script，style三个标签。不过也可以包含其他的自定义块。

#### 举例

```html
<template>
	<div class="example">
        {{ msg }}
    </div>
</template>

<script>
export default {
    data() {
        return {
            msg: "hello world!"
        }
    }
}
</script>

<style>
    .example {
        color: red;
    }
</style>

<custome1>
	这里的内容，比如可以包含对本组件的文档说明。
</custome1>
```

#### 解析: vue-loader

.vue单文件组件通过一个webpack的loader来解析，也就是vue-loader。这个loader会抽取里面的各个语言块，如果必要的话，会以管道的方式传递给其他的loader进一步处理，最终生成的是一个ES模块，这个模块默认export的是vuejs组件的选项对象。

vue-loader，还支持在各个语言块中使用lang属性指定其他的的语言，比如：

- html可以通过pug语言：```<template lang="pug">```
- css可以使用less, sass等等： ```<style lang="sass">```
- js可以使用typescript, coffeescript等。



### VUE插件

#### 什么是vue插件？

vue的插件就是一个包含install方法的对象，或者直接是一个函数。使用插件时，就会调用这个方法或者函数，这个方法或者函数接受两个参数，一个是vue app，一个是用户提供的options对象。

_插件的install方法只会被调用一次_。

如下是一个简单的编写插件的例子，使用的是包含install方法的对象：

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = key => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }
  }
}
```

使用插件的示例如下：

```js
import { createApp } from 'vue'
import i18n from './plugins/i18n'

const i18nStrings = {
  greetings: {
    hi: 'Hallo!'
  }
}

const app = createApp({})
app.use(i18n, i18nStrings) // 通过应用的use方法，就会调用install方法
```



#### 插件能干什么？

通过插件，可以给vue添加全局级功能，比如：

- 添加全局的方法或者property，比如：vue-custom-element。
- 添加全局的资源：directive/filter/animation，比如：vue-touch。
- 通过全局的Mixin来添加一些组件选项，比如：vue-router。
- 添加全局的实例方法，如给app.config.globalProperties添加属性。
- 一个库，提供自己的API，同时提供上面的一个或多个功能，比如：vue-router。



因为使用插件时，会传入app参数调用install函数，那么在install函数里，就可以使用app做很多事情：

1. app.directive()
2. app.provide()
3. 给app.config.globalProperties增加属性
4. app.mixin()



### VUE指令:directive

vue提供了很多内置的指令，这些指令用在html标签上或自定义组件标签上，实现一定的功能。如：

v-text：```<span v-text="msg"></span>``` 等价于 ```<span>{{msg}}</span>```

v-html：更新原始的innerHTML.

v-for：循环

v-if/v-else-if/v-else：实现按条件显示

v-show：根据条件，改变元素的display css属性

v-bind：可以缩写为：，绑定html属性，单向绑定

v-on：可以缩写为@,  用来注册事件，可以添加修饰符：stop, prevent, capture等等。

v-slot：可以缩写为#，实现插槽

v-model： 实现双向绑定，只用在input, select, textarea, components等输入控件。

v-pre：里面的内容，不会被编译，按原样显示，可以加快编译。

v-once：只渲染元素和组件一次。

v-cloak：



#### 自定义指令

有两种方式来自定义：

1. 自定义全局指令，使用app.directive()，指定一个指令定义对象，指令可以在任意组件中使用。
2. 自定义局部指令，在定义组件的选项中使用directives属性，指令只在当前组件中使用。

自定义指令的原理，就是通过指定一个对象，里面包含一系列钩子函数，然后vue会在特定的时机调用这些钩子函数，从而实现想要的功能。这些钩子函数包括如下七个：created, beforeMount, mounted, beforeUpdated, updated, beforeUnmount, unmounted。



这些钩子函数被调用是，会传入4个参数：

1. el： dom的element，可以直接操作dom元素。

2. binding: 使用时，给自定义指令传递的参数，有binding.arg, binding.value。如：

   ```<p v-pin:[direction]="200"></p>```，对于自定义指令pin来说，binding.arg=direction, binding.value=200。binding提供的参数还有：

   - instance: 使用指令的组件实例vm
   - oldValue: 先前的值，在beforeUpdate, updated钩子中可用。
   - modifiers：包含修饰符的对象，如v-pin.foo.bar对应的修饰符对象为{"foo":true, "bar": true}
   - dir: 注册指令时提供的指令定义对象。

3. vnode: 上面el对应的虚拟节点。

4. preNode: 上一个虚拟节点，在beforeUpdate, updated钩子中可用。



# vue-cli笔记

### VUE CLI包含有哪些内容

Vue CLI是一个完整的系统，包含如下组件：

- @vue/cli，是一个全局安装的npm包，提供了vue可执行程序，包含许多的命令，如：

  1. vue create: 使用命令行交互地创建一个vue项目
  2. vue ui: 打开一个创建vue项目的ui界面，可视化地创建项目。
  3. vue add:  给项目安装插件，并且调用插件的generator。
  4. vue upgrade: 用来升级vue的cli service，插件

  更详细的命令使用vue -h查看。

- @vue/cli-service，使用vue create创建的项目，会把它作为开发环境的依赖本地安装。它是构建在webpack和webpack-dev-server之上，包含：

  1. 核心服务，用来加载其他的CLI插件
  2. 内置的经过了优化的webpack配置，可以适用于大多数应用
  3. vue-cli-service的可执行程序，提供了serve, build, inspect等命令

- CLI插件

  CLI插件就是npm的包，给你的项目提供一些可选的特性，比如：Babel/TypeScript转译，ESLint的集成，单元测试，e2e测试等。插件命名遵循如下规范：

  1. 内置插件：@vue/cli-plugin-xxx，带vue的scope
  2. 社区第三方插件：vue-cli-plugin-xxx，带vue的前缀

  运行vue-cli-service可执行程序时，会自动解析加载所有在package.json中列出的CLI插件。插件可以在创建项目时添加，也可以创建完项目后添加。插件可以分组作为可以重用的预设（presets）。



### CLI plugins

vue CLI使用的是基于插件的架构。插件可以修改webpack的内部配置，给vue-cli-service可执行程序注入新的命令。

基于插件架构，使得vue CLI非常灵活和可扩展，你可以开发自己的插件。

#### 插件的组成

每个CLI插件都包含一个generator和一个runtime plugin。

- generator的作用：在你安装插件的时候会调用一次，用来创建文件。
- runtime plugin作用：会调整webpack的配置，注入新的命令给vue-cli-service。

#### 安装插件

在创建项目的时候，会根据你的选择安装一些插件。如果在项目创建以后还需要添加插件，可以使用vue add来安装。vue add 是专门用来安装CLI插件的，所以使用vue add安装插件时，可以省略插件的前缀cli-plugin-，同时优先从@vue scope下找插件，如：

```shell
vue add eslint
# 等价于下面, eslint存在于@vue，可以省略@vue
vue add @vue/eslint
# 等价于下面, 可以省略插件前缀
vue add @vue/cli-plugin-eslint

# 对于第三方插件，如下
vue add apollo
# 上面这个apollo插件不存在于@vue中，所以认为是第三方插件，会加上第三方插件的前缀，因此等价于下面：
vue add vue-cli-plugin-apollo

# 如果第三方插件也包含scope，那么必须带上scope, 如：
vue add @winson/{pluginNmae}
# 等价于
vue add @winson/vue-cli-plugin-{pluginNmae}
```



### Presets

#### 预设及其作用

一个vue CLI的预设，就是一个json文件，里面包含了预定义的一些选项和插件。创建新项目的时候就可以使用这个预设，而不需要在命令行提示符下进行选择了。在创建项目最后，可以把该次创建过程中使用的选项保存起来，就生成了一个预设。这个文件保存在用户home目录下的.vuerc文件中。可以直接编辑这个文件对预设的选项进行调整，新增和删除。

一个预设的例子：

```json
{
  "useConfigFiles": true, // 使用独立的配置文件保存webpack配置，而不是保存在package.json中
  "cssPreprocessor": "sass", // 使用sass作为css的预处理器
  "plugins": { // 插件及其插件的配置
    "@vue/cli-plugin-babel": {},
    "@vue/cli-plugin-eslint": {
      "config": "airbnb",
      "lintOn": ["save", "commit"]
    },
    "@vue/cli-plugin-router": {
      "prompts": true // 如果配置这个为true，表示让用户自己在命令提示符下选择插件的配置。
    },
    "@vue/cli-plugin-vuex": {},
    "vue-cli-plugin-winson": {
      "version": "^3.0.0" // 对于第三方的插件，最佳实践是显式地指定版本或者版本范围
    }
  },
  "configs": { // 也可以包含一些额外的集成工具的配置
    "vue": {...},
    "postcss": {...},
    "eslintConfig": {...},
    "jest": {...},
    "xxx": {...},
  }
}
```

这些额外的集成工具的配置，会被合并到package.json中，或者对应的独立的配置文件中（如果useConfigsFile:true），比如vue的配置保存到vue.config.js。



#### 预设的版本和命令提示

在预设的插件部分，通过version来指定插件的版本。对于官方的插件，你可以不指定版本，vue cli会使用最新的版本。对于第三方的插件，最好指定一个版本。



CLI插件一般都会有命令提示，创建项目时，会显示命令提示，让用户选择相应的选项。如果在预设中设置了插件的配置信息，那么这个命令提示就不会显示。这种情况下，你可以在预设中只声明要使用的插件，配置prompts为true来让插件显示命令提示。如：

```json
{
  "plugins": {
    "@vue/cli-plugin-eslint": {
      // 让用户来选择它自己的eslint配置
      "prompts": true
    }
  }
}
```

#### 远程预设

可以通过git库把一个预设发布出来，实现远程共享，让别人也可以使用。git库包含如下内容：

- preset.json   包含预设数据的主文件
- generator.js    一个generator，可以用来给项目中新建或者修改文件
- prompts.js  能够通过命令行提示符从用户处收集信息，给到generator。

一旦远程预设发布好了，用户创建项目时，可以通过--preset指定一个远程预设

```shell
vue create --preset remote-url hello-world
```

#### 本地文件系统预设

使用本地的json文件作为预设，可以避免预设改变时，频繁的push到远程仓库中。

```shell
# 使用绝对或者相对路径, 路径下存在预设文件，必须命名为preset.json
vue create --preset ./my_preset my_project

# 或者直接指定一个json文件，文件命名就随意了
vue create --preset my-preset.json my-project
```



### CLI Service

使用vue cli创建的项目，会安装@vue/cli-service的npm包，这个包包含一个vue-cli-service的可执行程序。

这个可执行程序可以直接在package.json中的scripts中使用，或者在命令行终端通过 ```./node_modules/.bin/vue-cli-service```直接调用。

事实上，使用vue cli创建的项目，默认就在package.json中的scripts中使用了这个可执行文件：

```json
{
  "scripts": {
    "serve": "vue-cli-service serve",
    "build": "vue-cli-service build"
  }
}
```

可以通过npm run启动执行。

也可以通过npx指定包，然后执行：

```shell
npx vue-cli-service serve
```

vue-cli-service除了serve, build, inspect，help这些自带的命令外，有些插件也可以给它注入新的命令。

比如```@vue/cli-plugin-eslint```就注入了lint命令。

可以使用help命令列出所有自带或者注入的命令。



可以通过--skip-plugins可以在执行命令时，跳过指定的插件，如：

```shell
npx vue-cli-service build --skip-plugins pwa,apollo
```



#### 缓存和并行执行

- 对于Vue/Babel/TypeScript的编译默认都开启了cache-loader。相关文件会被缓存到node_modules/.cache文件夹中，如果编译遇到问题了，始终先删除这个缓存目录试试。
- Babel/TypeScript的编译在多核处理器的机器上会开启thread-loader，来并行执行。

#### Git Hooks

@vue/cli-service包安装时，会默认安装上yorkie包，这个包允许你简单地在package.json文件中定义gitHooks属性，来指定git的钩子。比如在git commit前应该做些什么额外的操作。如：

```json
{
  "gitHooks": {
    "pre-commit": "lint-staged" // commit前，对于.js, .vue文件执行lint，然后git add
  },
  "lint-staged": {
    "*.{js,vue}": [
      "vue-cli-service lint",
      "git add"
    ]
  }
}
```

### CLI的配置

#### 全局配置

全局配置保存在home目录下，文件名为.vuerc，内容为json格式。可以指定如下内容：

- 你所偏爱的包管理器，如：npm,  yarn等。
- 本地保存的预设信息。
- 是否使用额外独立的配置文件。

#### 目标浏览器配置



#### vue.config.js

这个为项目级别的配置文件，存在的话，会被@vue/cli-service加载。你也可以在package.json中使用vue字段来指定项目级别的配置，此时你当然就只能使用json的格式了。所以使用vue.config.js的方式功能更强大，更灵活。vue.config.js需要导出一个对象，包含的属性有：

##### publicPath

默认为 '/'，在vue CLI3.3之前使用baseUrl这个属性，它表示应用部署的基础URL。

和webpack的publicPath意思一样，表示应用的上下文根。但是你应该一直使用publicPath而不是使用webpack的output.publicPath。

##### outputDir

默认为dist，表示构建时的bundle输出位置。应该一直使用这个配置而不是使用webpack的output.path。

##### assetsDir

默认为''，是相对outputDir的一个相对路径，表示静态资源的输出路径。

##### indexPath

默认值为'index.html'，index.html的输出路径，相对outputDir的相对路径。

##### filenameHashing

默认为true，输出的静态资源文件名中是否包含hash值。

##### pages

没有默认值，配置类型为一个对象。用来配置多页应用每个页面的配置，比如：

```js
module.exports = {
  pages: {
    index: {
      // 该页面的主入口
      entry: 'src/index/main.js',
      // 源html模板
      template: 'public/index.html',
      // 输出的页面html文件名： dist/index.html
      filename: 'index.html',
      // when using title option,
      // template title tag needs to be <title><%= htmlWebpackPlugin.options.title %></title>
      title: 'Index Page',
      // chunks to include on this page, by default includes
      // extracted common chunks and vendor chunks.
      chunks: ['chunk-vendors', 'chunk-common', 'index']
    },
    // when using the entry-only string format,
    // template is inferred to be `public/subpage.html`
    // and falls back to `public/index.html` if not found.
    // Output filename is inferred to be `subpage.html`.
    subpage: 'src/subpage/main.js'
  }
}
```

##### lintOnSave

默认为'default'，可配置为：boolean, 'warning', 'default', 'error'中的某个值。它被@vue/cli-plugin-eslint插件使用，用来指示如何处理代码异常情况。

##### runtimeCompiler

默认为false，表示是否使用Vue的核心构建功能，它包含一个运行时的编译器。如果为true的话，那么你可以在Vue的组件中使用template选项，不过此时你的应用体积会增大10Kb左右。

##### transpileDependencies

值为数组类型，默认为[]。它会被babel-loader使用，表示需要转译哪些位于node_modules中的依赖包。默认情况下，babel-loader是会忽略位于node_modules下的所有依赖，如果你确实需要转译它们，那么就可以使用这个配置显式地指定。

##### productionSourceMap

默认为true，表示是否在生产环境时，生成sourcemap。

##### crossorigin

没有默认值，html-webpack-plugin插件会使用这个配置来决定：在生成的html文件中过程中，如何设置link标签和script标签的crossorigin属性。它只影响插件本身生成的标签，用户在html模板中写的标签不会改动。

##### configureWebpack

类型为对象或者函数，如果为对象，vue会使用webpack-merge把它合并到webpack的配置中。

如果为函数，它会被调用，传入一个webpack的配置对象。你可以在函数中对配置对象进行修改等操作。

##### devServer

配置webpack-dev-server的信息，比如端口，主机名，代理等等。

##### pwa

配置pwa信息

#### babel.config.js

使用这个文件来配置Babel，这个Babel 7使用的配置方式，和.babelrc，package.json中的babel字段配置方式有点不一样。具体查看相关文档。

#### ESLint

ESLint可以使用package.json中的eslintConfig字段配置，也可以使用.eslintrc文件配置。

具体看@vue/cli-plugin-eslint插件的文档。

#### TypeScript

TypeScript通过tsconfig.json来配置，具体详情查看@vue/cli-plugin-typescript插件文档。

#### 单元测试

##### Jest

查看@vue/cli-plugin-jest插件文档

##### Mocha

查看@vue/cli-plugin-unit-mocha文档

#### 端到端测试

查看vue的官方端到端测试插件：Cypress, Nightwatch, WebdriverIO



### 开发vue CLI插件

