# Webpack 5学习笔记

### 相关概念

Webpack是一个静态模块打包器，包含一些核心概念：entry points, loader, plugin, output, mode。

- entry points：入口点，Webpack打包时的入口，以入口为起点，找到所有的依赖模块，进行打包。
- loader：打包时，通过不同的loader对不同模块类型进行加载，转换成JavaScript代码，打包对最终的bundle中，比如：html，css，img，js等。
- output：打包时，指定最终打包结果的输出路径和文件名称。
- plugin：能够执行比loader更广泛的功能，比如对打包过程进行优化，资源文件管理和注入环境变量等。
- mode：打包的模式，有production, development, none三种，不同的模式webpack采用不同的优化代码优化级别进行打包。



### 资源文件管理

Wepack天然只支持JavaScript和JSON的打包，其它的资源文件包含还包括：CSS,  图片，字体，CSV，XML等等。Webpack提供了对这些资源进行打包的方法。

#### 通过loader打包CSS资源

通过指定不同的loader实现：

```javascript
{
    module:{
        rules: [
            {
                test: /\.css$/i,
                use: ["style-loader", "css-loader"]
            }
        ]
    }
}
```

#### 通过内置的资源模块打包图片，字体资源

Webpack5之前可以通过一些loader对图片，字体资源打包。Webpack5引入了资源模块的概念。在rules的配置项中，增加了type选项来指定资源的类型，无须指定具体的loader，也可以实现对图片，字体的打包。当然也支持旧的通过loader的方式，type有5种选择：

- asset/resource: 生成单独的文件，导出一个URL。以前要通过file-loader实现。
- asset/inline: 导出资源的数据URL。以前通过url-loader实现。
- asset/source: 导出资源文件中的原始数据内容。以前要通过raw-loader实现。
- asset: 也就是在asset/resource和asset/inline中自动选择一个。以前通过带size limit选项的url-loader实现。
- javascript/auto: 如果要使用老的file-loader，url-loader, raw-loader的话，type可以指定为这个值。

举例如下：

```javascript
// 使用了旧的指定loader的方式，此时type为: javascript/auto
module.exports = {
  module: {
   rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
            }
          },
        ],
       type: 'javascript/auto'
      },
   ]
  },
}

// 通过type指定资源类型，由Webpack5内置的资源模块进行打包
// 可以通过output.assetModuleFilename统一指定所有资源文件打包后的目标文件名模式
// 也可以通过在具体的rule中指定generator.filename来指定某类资源文件打包后的目标文件名模式
module.exports = {
 output: {
   filename: 'main.js',
   path: path.resolve(__dirname, 'dist'),
   // 全局指定
   assetModuleFilename: 'images/[hash][ext][query]'
  },
 module: {
   rules: [
     {
       test: /\.png/,
       type: 'asset/resource'
     },
     {
       test: /\.html/,
       type: 'asset/resource',
       // 局部指定
       generator: {
         filename: 'static/[hash][ext][query]'
       }
     }
   ]
 },
};
```

#### 打包数据资源

数据资源有JSON files, CSVs, TSVs, and XML等，JSON已经天然就支持了。可以通过csv-loader对CSV，TSV文件，xml-loader对XML文件进行打包。

如：

```javascript
 module.exports = {
   module: {
     rules: [
      {
        test: /\.(csv|tsv)$/i,
        use: ['csv-loader'],
      },
      {
        test: /\.xml$/i,
        use: ['xml-loader'],
      },
     ],
   },
 };
```

处理使用loader外，可以使用自定义的解析器，来把toml，yaml，json5格式文件，解析成JSON数据。这些自定义解析器，已经被实现了出来了，只需按照并使用即可。

```shell
# 安装
npm i -D toml, yamljs, json5
```

```javascript
// 使用
const toml = require('toml');
const yaml = require('yamljs');
const json5 = require('json5');
module.exports = {
    module: {
        rules: [
           {
               test: /\.toml$/i,
               type: 'json',
               parser: {
                   parse: toml.parse,
               },
           },
            {
                test: /\.yaml$/i,
                type: 'json',
                parser: {
                    parse: yaml.parse,
                },
            },
            {
                test: /\.json5$/i,
                type: 'json',
                parser: {
                    parse: json5.parse,
                },
            },
        ],
    },
};
```



### 打包输出管理

#### output.filename

指定输出的主包的文件名模式

#### output.path

指定文件的输出路径

#### 自动生成主页面文件

使用WebpackHtmlPlugin插件，自动生成页面html，并且里面的css和js已经自动引入好了。



### 开发环境配置



### 代码分割

有三种常用的代码分割的方式：

- 通过多个entry points
- 防止重复：使用入口点依赖和SplitChunksPlugin
- 动态导入

#### 使用多个入口点

这种分割代码的方式是纯手工的，最简单直观，针对每个入口点，都会打包出一个bundle。这种方式不会去重，会导致重复的代码在bundle中。

```javascript
module.exports = {
    // 这种方式会生成两个bundle，分割了代码，如果两个代码都引入了lodash
    // 最终的index.bundle.js和print.bundle.js中都会包含lodash的代码，造成了代码重复的问题。
    entry: {
        index: "./src/index.js",
        print: "./src/print.js"
    },
    output: {
        filename: "[name].bundle.js",
        path: resolve(__dirname, 'dist'),
        clean: true
    }
}
```

#### 防止重复

##### 入口依赖

使用dependOn选项在入口中指定共同的依赖





