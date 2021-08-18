# DOM level 3事件流规范

### 简介

本文简单描述了事件的派发机制，以及事件是怎么在DOM树上传播的。

#### 事件由谁产生？

一般产生事件的方式有：

1. 用户触发：如键盘，鼠标事件。
2. 由API产生：如指示动画已经完成运行，dom已经加载完成，视频已经被暂停等。
3. 也可以有脚本产生：如调用HTMLElement.click()，或者EventTarget.dispatchEvent()

#### 产生的事件如何表示？

产生的事件是实现了Event接口的对象。不同方式产生的事件，事件对应的对象是不同的，不过都会实现Event接口。

#### 产生的事件如何处理？

很多DOM元素可以监听这些事件，给DOM元素添加事件监听处理函数方式有：

1. EventTarget.addEventListener
2. 在HTML标签的属性上绑定，如：<button onclick="">







### 参考





