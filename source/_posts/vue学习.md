---
title: vue学习
date: 2019-03-18 22:15:05
tags:
---

#### mvvm

* Model   模型，数据对象(data)

* view  视图模板页面

* viewModel   视图模型(vue的实例)

  <!--more-->

#### 表达式和指令

1. “Mustache”语法 (双大括号) 的文本插值

   ```html
   <span>Message: {{ msg }}</span>
   ```

2. Mustache 语法不能作用在 HTML 特性上，遇到这种情况应该使用 v-bind 指令：

   ```html
   <div v-bind:id="dynamicId"></div>
   <div :id="dynamicId"></div>  <!--简写形式-->
   <!--在布尔特性的情况下，它们的存在即暗示为true-->
   <button v-bind:disabled="isButtonDisabled">Button</button>
   ```

3. 使用js表达式

   ```html
   {{ number + 1 }}
   {{ ok ? 'YES' : 'NO' }}
   {{ message.split('').reverse().join('') }}
   <div v-bind:id="'list-' + id"></div>
   ```

4. 事件绑定

   ```html
   <a v-on:click="doSomething">...</a>
   <!-- 修饰符  .prevent 修饰符告诉 v-on 指令对于触发的事件调用 event.preventDefault()-->
   <form v-on:submit.prevent="onSubmit">...</form>
   <!-- 缩写 -->
   <a @click="doSomething">...</a>
   <!--传参 -->
   <a @click="doSomething('abc')">...</a>
   ```

#### 计算属性和监视

1. 计算属性

   ```js
   var vm = new Vue({
     el: '#example',
     data: {
       firstName: 'Hello',
       lastName:'123'
     },
     computed: {
       // 计算属性的 getter 方法的返回值作为属性值
       //什么时候执行：初始化显示、相关data属性发生改变
       fullName() {
         // `this` 指向 vm 实例
         return this.firstName +' '+ lastName
       }
     }
   })
   //计算属性的 setter
   computed: {
     fullName: {
       // getter  读取当前属性值时根据相关的数据回调
       //计算属性存在缓存
       get: function () {
         return this.firstName + ' ' + this.lastName
       },
       // setter  当属性值发送改变时回调
       set: function (newValue) {
         var names = newValue.split(' ')
         this.firstName = names[0]
         this.lastName = names[names.length - 1]
       }
     }
   }
   ```

2. 监视

   ```js
   var vm = new Vue({
     el: '#demo',
     data: {
       firstName: 'Foo',
       lastName: 'Bar',
       fullName: 'Foo Bar'
     },
     watch: {
       firstName: function (val) {
         this.fullName = val + ' ' + this.lastName
       },
       lastName: function (val) {
         this.fullName = this.firstName + ' ' + val
       }
     }
   })
   ```

3. class与style绑定

   * class绑定  class='xxx'

     xxx 字符串 

     xxx 对象  （用的比较多）

     ```html
     <div v-bind:class="{ active: isActive }"></div>
     ```

     xxx 数组

   * style绑定

     ```html
     <div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
     ```

     ```js
     data: {
       activeColor: 'red',
       fontSize: 30
     }
     ```

4. 条件渲染

   

