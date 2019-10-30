---
title: js高级程序设计-读书笔记2
date: 2018-05-04 22:37:24
tags:
    - js高级程序设计
categories:
    - 读书笔记
---
### 引用类型
1. Object类型
    创建Object实例的方式有两种：
    
    ```js
     var person = new Object();
       person.name = "Nicholas";
       person.age = 29;
       
       //字面量方式:
       var person = {
           name : "Nicholas",
           age :29
       }
    ```
    
    <!--more-->

一般来说，可以使用点表示法来访问对象属性，但也可以使用[]来访问对象属性，应该将要访问的属性以字符串的形式放在[]中，[]方法的主要优点是可以通过变量来访问属性。
2. Array类型 
   1. 创建方式：
   
   ```js
      var colors = new Array();
      var colors = new Array(20);
      var colors = new Array("red","blue","green");
      var colors = new Array(3);//创建一个包含三项的数组
      var colors = new Array("Greg");//创建包含"Greg"一项的数组
      var colors = Array();//new 可以省略
      var colors = [];
      var colors = ["red","blue","green"]
      var colors = ["red","blue",]//不要这样，这样会创建一个包含两个或三个的数组
      var colors = [,,,,,] //不要这样，这样会创建一个包含五个或六个的数组
      
      var colors = ["red","blue","green"];
    colors.length = 2;
    alert(colors[2]) ; //undefined
      
    var colors = ["red","blue","green"];
    colors.length = 4;
    alert(colors[3]); //undefined
      
    //利用length可以很方便的在数组末尾添加新项:
    var colors = ["red","blue","green"];
    colors[colors.length] = "black";
    colors[colors.length] = "brown";
    
   ```
  
3. 数组的每一项都可以保存任何类型的数据。
4. 检测数组:    

    ```js
    if(value instanceof Array){
        //对数组执行某些操作
    }
    ```
    

instanceof操作符的问题在于，它假定只有一个全局执行环境。如果网页中包含多个框架，那实际上就存在两个以上不同的全局执行环境，从而存在两个以上不同版本的Array构造函数。如果从一个框架向另一个框架传入数组，那么传入的数组与第二个框架原生创建的数组分别具有各自不同的构造函数，要解决这个问题，es5新增了Array.isArray()方法，这个方法的目的是最终确定某个值到底是不是数组，而不管他是在哪个全局执行环境中创建的。 

```js
if(Array.isArray(value)){
    //对数组执行某些操作
}
```

5. 转换方法

所有对象都具有toLocalString()、toString()、valueOf()方法。调用数组的toString()方法会返回有数组中每个值的字符串形式拼接而成的一个以逗号分隔的字符串。而调用valueOf()返回的还是数组。实际上，为了创建这个字符串会调用数组每一项的toString()方法。
如果数组中的某一项的值是null、undefined,那么该值在join()、toLocalString()、toString()、valueOf()返回的结果中以空字符串表示。

6. 栈方法

   1. push()可以接收任意数量的参数，把他们逐个添加到数组的末尾，并返回修改后数组的长度
   2. pop()方法从数组末尾移除最后一项，减少数组的length值，返回移除的项
7. 队列方法

   1. 由于push()是向数组末尾添加项的方法，因此要模拟队列只需一个从数组前端取得项的方法。实现这一操作的方法是shift(),他能够移除数组中的第一个项并返回该项，同时将数组长度减一。结合shift()和push()方法，可以像使用队列一样使用数组。
   2. unshift()与shift()用途相反，它可以在数组前端添加任意项并返回新数组的长度。因此，同时使用unshift()和pop()方法，可以从相反的方向来模拟队列
   
8. 排序

   1. sort()方法会调用每个数组项的toString()转型方法，然后比较得到的字符串，以确定如何排序。即使数组中的每一项都是数值，sort方法比较的也是字符串，如下：
      
      ```js 
      var values = [0,1,5,10,15]
      values.sort()
      alert(values)  //0,1,10,15,5
      ```
      
    2. sort()方法可以接收一个比较函数作为参数，以便我们指定那个值位于那个值得前面。比较函数接收两个参数，如果第一个参数应该位于第二个之前则返回一个负数，如果两个参数相等则返回0，如何第一个参数应该位于第二个之后则返回一个正数。
       
        ```js
        function compare(v1,v2){
            if(v1 < v2){
                return 1;
            }else if(v1>v2){
                return -1;
            }else{
                return 0;
            }
        }
        
        values.sort(compare); // 15,10,5,1,0
        
        //对于数值类型或者valueOf()方法会返回数值类型的对象类型，可以使用一个更简单的比较函数
        function compare(v1,v2){
            return v1 - v2;
        }
        ```
   
9. 操作方法

   1. concat()该方法会先创建当前数组一个副本，然后将接收到的参数添加到这个副本的末尾，最后返回新构建的数组。在没有给concat()传递参数的情况下，它只是复制当前数组并返回副本。如果传递给concat()方法的是一个或多个数组，则该方法会将这些数组中的每一项都添加到结果数组中。如果传递的值不是数组，这些值就会简单的添加到结果数组的末尾。例如：
      
     ```js
     var colors = ['red','green','blue'];
     var colors2 = colors.cancat('yellow',['black','brown']);
     alert(colors) //red,green,blue
     alert(colors2) //red,green,blue,yellow,black,brown
     ```
   
  2. slice()基于当前数组中的一个或多个项创建一个新数组。slice()方法接收一或两个参数，即要返回项的起始和结束位置。在只有一个参数的情况下，slice()返回从该参数指定位置开始到当前数组末尾的所有项。如果有两个参数，该方法返回起始和结束位置之间的项，但不包括结束位置的项。slice()方法不会影响原始数组。
  3. splice()
  
        * 删除：可以删除任意数量的项，只需指定2个参数：要删除的第    一项的位置和要删除的项数
        * 插入：可以向指定位置插入任意数量的项，只需提供三个参数：起始位置、0(要删除的项数)和要插入的项。如果要插入多个项，可以再传入第四、第五，一直任意多个项。
        * 替换：可以向指定位置插入任意数量的项，且同时删除任意数量的项，只需指定3个参数：起始位置、要删除的项数和要插入的任意数量的项。插入的项不必与删除的项数相等。
        
        ```js
        var colors = ['red','green','blue'];
        var removed = colors.splice(0,1);//删除第一项
        alert(colors) //green,blue
        alert(removed) //red
        
        removed = colors.splice(1,0,'yellow','orange');//从位置1开始插入两项
        alert(colors) //green,yellow,orange,blue
        alert(removed) //返回的是一个空数组
        
        removed = colors.splice(1,1,'red','purple')
        alert(colors) //green,red,purple,orange,blue
        alert(removed) // yellow
        ```
        
10. 位置方法:

    1. indexOf()从数组的开头开始向后查找
    2. lastIndexOf()从数组的末尾开始向前查找
    3. 这两个方法都返回要查找的项在数组中的位置，在没有找到的情况下返回-1。在比较第一个参数和数组中的每一项时，会使用全等操作符
    
11. 迭代方法：每个方法都接收两个参数：要在每一项上运行的函数和(可选的)运行该函数的作用域对象--影响this的值。传入这些方法的函数会接收三个参数：数组项的值，该项在数组中的位置和数组对象本身。

    1. every():对数组中的每一项运行给定函数，如果该函数对每一项都返回true,则返回true.
    2. filter():对数组中的每一项运行给定函数，返回该函数会返回true的项组成的数组。
    3. forEach():对数组中的每一项运行给定函数,没有返回值。
    4. map():对数组中的每一项运行给定函数,返回每次函数调用的结果组成的数组。
    5. some():对数组中的每一项运行给定函数,如果该函数对任一项返回true,则返回true。
    
    ```js
    var numbers = [1,2,3,4];
    var everyResult = numbers.every(function(item,index,array){
        return (item >2)
    })
    ```
    
12. 归并方法：这两个方法都接收两个参数：一个在每一项上调用的函数和(可选的)作为归并基础的初始值。传给reduce()和reduceRight()的函数接收四个参数：前一个值、当前值、项的索引和数组对象
    1. reduce()迭代数组中的每一项，构建一个最终的返回值，从数组的第一项开始。
    2. reduceRight()从数组的最后一项向前遍历。
    
    ```js
    var values = [1,2,3,4,5];
    var sum = values.reduce(function(prev,cur,index,array(){
        return prev+cur;
     });
     alert(sum) //15
    ```
   ```
    
        
        
        
        
    



   ```