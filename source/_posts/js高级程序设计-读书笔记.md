---
title: js高级程序设计-读书笔记
date: 2018-04-30 21:40:38
tags:
    - js高级程序设计
categories:
    - 读书笔记
---
### `<script>`标签
1. 无论如何包含代码，只要不存在defer和async属性，浏览器都会按照`<script>`元素在页面中出现的先后顺序对他们依次进行解析。

2. 在文档的`<head>`元素中包含所有javascript标签，意味着必须等到全部的javascript代码都被下载、解析和执行完成后，才能开始呈现页面的内容（浏览器在遇到`<body>`标签时才开始呈现内容），所以，一般都这样写：

    <!--more-->
    
    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=<device-width>, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <!--这里放内容-->
        <script type="text/javascript" src="example1.js"></script>
        <script type="text/javascript" src="example2.js"></script>
    </body>
    </html>
    ```
    
3. defer属性：脚本会被延迟到整个页面都解析完毕后再运行。因此，在`<script>`元素中设置defer属性，相当于告诉浏览器立即下载，但延迟执行。defer属性只适用于外部脚本文件。在下面的代码中，虽然`<script>`放在了文档`<head>`元素中，但其中包含的脚本将延迟到浏览器遇到`</html>`标签后再执行。HTML5规范要求脚本按照他们出现的先后顺序执行，因此第一个延迟脚本会先于第二个延迟脚本执行，而这两个脚本会先于DOMContentLoaded事件执行。在现实中，延迟脚本并不一定会按照顺序执行，也不一定会在DOMContentLoaded事件触发前执行，因此最好只包含一个延迟脚本。

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=<device-width>, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
        <script type="text/javascript" defer="defer" src="example1.js"></script>
        <script type="text/javascript" defer="defer" src="example2.js"></script>
    </head>
    <body>
        <!--这里放内容-->
    </body>
    </html>
    ```

4. HTML5为`<script>`元素定义了async属性。async属性告诉浏览器立即下载文件，但并不保证按照他们的先后顺序执行。

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=<device-width>, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
        <script type="text/javascript" async src="example1.js"></script>
        <script type="text/javascript" async src="example2.js"></script>
    </head>
    <body>
        <!--这里放内容-->
    </body>
    </html>
    ```
    在以上代码中，第二个脚本可能会在第一个脚本之前执行，因此，确保两者之间互不依赖非常重要。指定async属性的目的是不让页面等待两个脚本下载和执行，从而异步加载页面其他内容。为此，建议异步脚本不要在加载期间修改DOM。

### 数据类型
1. 五种基本数据类型：Undefined、Null、Boolean、Number、String;一种复杂数据类型：Object。
2. 在使用var声明变量但未对其加以初始化时，这个变量的值就是undefined。

    ```
    var message;//这个变量声明之后默认取得了undefined值
    //下面这个变量并没有声明
    //var age
    alert(typeof message);  //"undefined"
    alert(typeof age);      //"undefined"
    ```
    对未声明和未初始化的变量执行typeof操作符都返回了undifined值。
3. 如果定义的变量准备在将来用于保存对象，那么最好将该变量初始化为null而不是其他值。这样一来，只要直接检查null值就可以知道相应的变量是否已经保存了一个对象的引用，如下：
   
    ```
    if(car != null){
        //对car执行某些操作
    }
    //实际上，undefined的值是派生自null值的
    alert(null == undefined) //true
    ```
4. NaN,即非数值(Not a Number)是一个特殊的数值，这个数值用来表示一个本来要返回数值的操作数未返回数值的情况（这样就不会抛出错误了）。任何数值除以0会返回NaN；任何涉及NaN的操作（NaN/10）都会返回NaN;NaN与任何值都不相等，包括NaN本身。

### 执行环境及作用域
　　执行环境(execution context)是javaScript中最为重要的一个概念。执行环境定义了变量或函数有权访问的其他数据，决定了他们各自的行为。每个执行环境都有一个与之关联的变量对象(variable object)，环境中定义的所有变量和函数都保存在这个对象中。虽然我们编写的代码无法访问这个对象，但解析器在处理数据时会在后台使用它。
　　全局执行环境是最外围的一个执行环境。根据宿主环境不同，执行环境的对象也不一样。在web浏览器中，全局执行环境被认为是window对象，因此所有全局变量和函数都是作为window对象的属性和方法创建的。
　　每个函数都有自己的执行环境。当执行流进入一个函数时，函数的环境就会被推入一个环境栈中。而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。
　　当代码在一个环境中执行时，会创建变量对象的一个作用域链(scope chain)。作用域链的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。作用域链的前端，始终都是当前执行的代码所在环境的变量对象。如果这个环境是函数，则将其活动对象(activation object)作为变量对象。活动对象在最开始时只包含一个变量，即arguments对象(这个对象在全局环境中是不存在的)。作用域链中的下一个变量对象来自包含环境，而再下一个变量对象则来自下一个包含环境。这样，一直延续到全局执行环境；全局执行环境的变量对象始终都是作用域链中的最后一个对象。    

  


​    

