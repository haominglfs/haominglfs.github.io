---
title: js原型
date: 2019-09-24 21:37:02
tags:
---

1. obj这个对象本质上是被Object函数创建的，因此`obj.__proto__=== Object.prototype`。我们可以用一个图来表示。

   ![](https://raw.githubusercontent.com/haominglfs/images/master/20190925203303.png)

    即，每个对象都有一个`__proto__`属性，指向创建该对象的函数的prototype。

2. 自定义函数的prototype本质上就是和 var obj = {} 是一样的，都是被Object创建，所以它的`__proto__`指向的就是Object.prototype。但是Object.prototype确实一个特例——它的`__proto__`指向的是null。

   ![](https://raw.githubusercontent.com/haominglfs/images/master/20190925204029.png)

3. 函数也是一种对象，函数是由`Function`，所以`Object.__proto__ === Function.prototype`

   ```javascript
   function fn(x,y){
     return x+y;
   }
   var fn = new Function('x','y','return x+y');
   ```

   

   









   





