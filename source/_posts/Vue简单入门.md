---
title: Vue简单入门
categories: VUE
tags:
  - VUE
description: Vue简单入门
toc: false
date: 2018-03-26 10:30:48
---

# 基本结构

+ template

**基本HTML语法**
+ script

**实例化的Vue对象**
```javascript
var sth = new Vue({});
```
+ css

**样式**
# Vue对象基本结构

+ data

**数据域**
+ methods

**用来定义功能函数**
+ watch

**用来定义数据监视函数**
**Demo如下:**
```javascript
new Vue({
    data:{
        a:1,
        b:[]
    },
    props: [ '注册组件名称' ]，
    methods:{
        doSomething:function() {
            /* 调用此函数时候打印**a**的值 */
            console.log(this.a)
        }
    },
    watch:{
        'a' : function(val,oldVal) {
            /* 当**a**的值发生改变时，调用该方法，打印**a**的旧值与新值 */
            console.log(val,oldVal)
        }
    }
})
```

# 模版指令

## 数据渲染

**v-text** && **v-html** && **{{ 变量 }}**
```html
<p>{{ a }}</p>
<p v-text="a"></p>
<p v-html="a"></p>
```
**脚本部分**
```javascript
new Vue({
    data : {
        a: 1
    }
})
```
## 模块显隐性
**v-if** && **v-show**
**区别**
**v-if**：为假时直接不渲染该块；
**v-show**：类似于CSS中display:none;
```html
<p v-if="isShow"></p>
<p v-show="isShow"></p>
```

**脚本部分**
```javascript
new Vue({
data : {
   isShow: true
}
})
```
## 渲染循环列表
**v-for**
```html
<ul>
    <li v-for="item in items">
        <p v-text="item.label"></p>
    </li>
</ul>
```

**脚本部分**
```javascript
new Vue({
data : {
    items : {
        {
            label: 'apple'
        },
        {
            label: 'banana'
        }
    }
}
})
```

## 事件绑定
**v-on** <==>  **@**
```html
<button v-on:click="doThis"></button>
<button @click="doThis"></button>
```

```javascript
    methods: {
        doThis: function(sth) {
        }
    }
```
## 属性绑定
**v-bind** <==> **:**
```html
<!-- imgSrc 为字符串 -->
<img v-bind:src="imgSrc">

<!-- red 表示名为 red 的CSS样式（.red），isRed是data域中的布尔值，根据真假确定该样式是否渲染 -->
<div :class"{ red : isRed }"></div>
<!-- 包含两种CSS样式（.classA && .classB） -->
<div :class"[classA, classB]"></div>
<!-- 组合 -->
<div :class"[classA, {classB: isB, classC: isC}]"></div>
```

# 组件化
## 组件的引入与注册
+ 引入组件后使用前需要注册，使用**components**

> 注册组件后使用时候将驼峰转为中线形式原因：
> ComponentA之所以要转换成component-a
> 因为在HTML中，标签的标签名是大小写不敏感的，而在javascript中变量名是大小写敏感的
> 换句话说，在HTML中，ComponentA和componenta算是同一个标签
> 而在javascript中，ComponentA和componenta却不是同一个变量
> 要解决这种差异问题，就必须得在两种标准之间做一个转换，所以vuejs就使用了将驼峰法(camelCase)转换到短横线法(kebab-case)

+ 父向子组件传值**props**传值(props与data，methods等同级)
+ 子向父组件传值**自定义事件**  ||  **\$children**  ||  **\$refs**


> - $on() 监听事件
> - $emit() 在他上面触发时间

**考虑到树结构的层次复杂性**
> - ~~$dispatch() 派发事件，事件沿着父链冒泡~~（VUE2.0已弃用）
> - ~~$broadcast() 广播事件，事件向下传到给所有后代~~（VUE2.0已弃用）