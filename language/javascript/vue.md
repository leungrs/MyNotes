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





