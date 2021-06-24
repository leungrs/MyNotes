### Gradle简介

gradle是一个开源的构建自动化工具。
- 高性能：按需构建；构建缓存；
- 基于JVM；
- 基于约定，也可以自定义；
- 可以扩展
- IDE支持：Android Studio，IDEA，Eclipse，NetBeans。

关于Gradle，必须知道的5件事：
1. Gradle是一个通用的构建工具。因为它对要构建的东西以及如何构建没有做任何的假设，可以用来架构任何软件。最大的限制就是目前的依赖管理只支持maven和ivy兼容的构件库和文件系统。
2. 核心模型是基于任务的，任务与任务之间的依赖关系形成一个有向无环图的结构。
    一个任务由三个部分组成：
    - 动作：比如拷贝文件，编译源代码
    - 输入：动作的操作对象，比如具体的值，文件或者目录
    - 输出：执行动作后的输出，比如生成一些文件或者目录
3. Gradle包含若干个固定的构建阶段，形成了Gradle的构建生命周期
    - 初始化：为构建设置环境，决定哪些项目参与到构建过程中
    - 配置：构造和配置任务图，然后决定哪些任务应该以何种顺序进行构建
    - 执行：按照配置的任务图，执行所选择的任务
4. Gradle是可扩展的：
    - 自定义任务类型：当你的构建过程存在一些已有的任务不能完成的功能时，可以编写自己的任务类型。最好是把自定义任务类型的源代码放到buildSrc文件夹或者一个打包好的插件里。
    - 自定义任务动作：通过Task.doFirst和Task.doLast方法，在任务执行的前后执行你自定义的逻辑。
    - 在项目和任务上使用Extra properties
    - 自定义约定：约定是一种可以简化构建，让用于易于使用和理解的强大途径，比如Java的构建约定。
    - 自定义模型：处理任务，文件，依赖配置外，Gradle允许你引入新的概念模型到构建过程中。你可以在大部分的语言插件中看到这种用法。
5. 构建脚本是基于API来操作的，可以把构建脚本当做可执行的代码看待，因为他们本来就是可执行代码。不过这都是实现的细节。真正设计良好的构建脚本描述了构建软件的需要的是哪些步骤，而不是这些步骤应该怎么来实现的，具体的实现是自定义任务类型或者插件要完成的。

    一个普遍的误解就是Gradle的强大和它的灵活性是因为它的构建脚本就是代码。其实这不是真相，真相在于它的底层模型和API。官方推荐的最佳实践是：你应该避免在你的构建脚本里放置太多（如果有的话）强制逻辑，构建脚本应该是声明式的。

    确实有一个地方，你把构建脚本当成可执行代码会很有帮助：那就是理解如何把构建脚本的语法影射到Gradle的API上。API文档（由Groovy DSL手册和Javadocs组成）列出了方法，属性，闭包和动作的参考资料。

    Gradle运行在JVM上，所有构建脚本可以使用标准的Java API，另外Groovy的构建脚本可以使用Groovy的API，Kotlin的构建脚本可以使用Kotlin的API。


