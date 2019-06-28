---
title: css module和scoped的区别
date: 2019-06-28 14:25:28
tags: [scoped, css module]
---

现为了解决vue中css作用域的问题，vue内置了两种解决方法：Scoped CSS 和 CSS Modules (模块式 CSS)。下面对它们进行详细的介绍。
<!--more-->
## scoped css

### 什么是scoped css

在Vue文件中的style标签上有一个特殊的属性，scoped。当一个style标签拥有scoped属性时候，它的css样式只能用于当前的Vue组件，可以使组件的样式不相互污染。如果一个项目的所有style标签都加上了scoped属性，相当于实现了样式的模块化。

### scoped原理：

Vue中的scoped属性的效果主要是通过PostCss实现的。以下是转译前的代码:
```
<style scoped lang="less">
    .example{
        color:red;
    }
</style>
<template>
    <div class="example">scoped测试案例</div>
</template>

```
转译后:

```
.example[data-v-5558831a] {
  color: red;
}
<template>
    <div class="example" data-v-5558831a>scoped测试案例</div>
</template>

```

既:PostCSS给一个组件中的所有dom添加了一个独一无二的动态属性，给css选择器额外添加一个对应的属性选择器，来选择组件中的dom,这种做法使得样式只作用于含有该属性的dom元素(组件内部的dom)。


scoped的渲染规则：

1.给HTML的dom节点添加一个不重复的data属性(例如: data-v-5558831a)来唯一标识这个dom 元素
2.在每句css选择器的末尾(编译后生成的css语句)加一个当前组件的data属性选择器(例如：[data-v-5558831a])来私有化样式


### scoped css的缺点

1.如果用户在别处定义了相同的类名，也许还是会影响到组件的样式。
2.根据css样式优先级的特性，scoped这种处理会造成每个样式的权重加重了：即理论上我们要去修改这个样式，需要更高的权重去覆盖这个样式。所以在引用包含scoped的第三方插件时如若需要修改样式则需要全局修改，而且要注意权重问题，将迫不得已再使用!important。
3.如果组件内部包含有其他组件，只会给其他组件的最外层标签加上当前组件的data属性：
所以一般父组件如果加了scoped，会比已经设置过自己样式的子组件内除最外层标签的内层标签的权重低，影响不到他们的样式。
即下图所示：

```
<template>
  <BasePanel class=”pricing-panel”>
    content  </BasePanel>
</template>
<style scoped>
  .pricing-panel {
    width: 300px;
    margin-bottom: 30px;
  }
</style>
```
转义后

```
<style>
  .base-panel[data-v-d17eko1] {
    ...
  }
  .pricing-panel[data-v-b52c41] {
    width: 300px;
    margin-bottom: 30px;
  }
</style>
<div class=”base-panel pricing-panel” data-v-d17eko1 data-v-b52c41>content</div>

```

不过也是可以通过如下方法影响到的：

![深度作用选择器](./deep.png)

4.scoped会使标签选择器渲染变慢很多倍

![scoped造成的性能影响](./xingneng.png)


## css module

css module需要增加css-loader配置才能生效，具体可看文档<https://vue-loader.vuejs.org/zh/guide/css-modules.html>的实现

转译前：
```
<template>
  <p :class="$style.gray">
    Im gray
  </p>
</template>

<style module>
.gray {
  color: gray;
}
</style>
```
转译后

```
<p class="gray_3FI3s6uz">Im gray</p>

.gray_3FI3s6uz {
    color: gray;
}
```

由此可见，css module直接替换了类名，排除了用户设置类名影响组件样式的可能性。


## 总结

其实两种方案都非常简单、易用，在某种程度上解决的是同样的问题。
scoped 样式的使用不需要额外的知识，给人舒适的感觉。它所存在的局限，也正是它的使用简单的原因。它可以用于支持小型到中型的应用。
在更大的应用或更复杂的场景中，这个时候，对于 CSS 的运用，我们就会希望它更加显式，拥有更多的控制权。虽然在模板中大量使用 $style 看起来并不那么“性感”，但却更加安全和灵活，为此我们只需付出微小的代价。还有一个好处就是我们可以用 JS 获取到我们定义的一些变量（如色彩值、样式断点），这样我们就无需手动保持其在多个文件中同步。

