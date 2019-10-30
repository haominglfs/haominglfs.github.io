---
title: es6
date: 2019-02-07 21:53:13
tags:
---

* 数据类型

  Number、Boolean、String、**Symbol**、Null、Undefined、Object

  Symbol:es6新增数据类型-基本类型，值是由Symbol函数调用产生的

  <!--more-->

* 作用域

  let、const

* 解构赋值

  * 数组解构

    let [a,b,c] = [1,2,3]

  * 对象解构

    let {foo,bar}  = {foo:"aaa",bar:"bbb"};//左侧变量的名称解构顺序无关

    let {foo:f,bar:b} = {foo:"aaa",bar:"bbb"};//取别名

    let {foo:[a,b]} = {foo:[10,20],bar:"bbb"}//多重解构

* 扩展运算符  ...

  把数组、对象转成参数序列。

  var arr = [1,7,3,6,5];

  var arr2 = ['a','b','c'];

  Math.max(...arr);

  var arr3 = [...arr,...arr2];

* 模板字符串（保持格式、表达式解析）

  反引号   ${}

* 对象简洁表示法

  当对象的key与对应的属性所引用的变量或函数同名的时候，可以简写成一个。

  ```javascript
  var a = 1;
  var fn = function(){}
  
  var obj = {
      a:a,
      fn:fn
  }
  
  //简写方式
  var obj = {
      a:,
      fn
  }
  ```

* 属性名表达式

  ```javascript
  var x = 'hh'
  var obj = {
      [x]:1  //key可以为动态的
  }
  ```

* 迭代

  ```javascript
  var attrs = ['a','b','c'];
  for(var attr in attrs){
      console.log(attr)  // 0 1 2
  }
  
  for(var attr of attrs){
      console.log(attr)  // a b c
  }
  ```

  迭代器

  ```javascript
  var obj = {
              left :100,
              top :200
          }
  
  obj[Symbol.iterator] = function(){
      let keys = Object.keys(obj) //['left','top']
      let len = keys.length;
      let n = 0;
      return {
          next:function(){
              if(n<len){
                  return{
                      value:{k:keys[n],v:obj[keys[n++]]},
                      done:false
                  }
              }else{
                  return{
                      done:true
                  }
              }
          }
      }
  }
  
  for(var {k,v} of obj){
      console.log(k,v)
  }
  ```

* 函数扩展

  1. 参数默认值

     ```javascript
     function fn2(x=0,y=2){}
     ```

  2. rest参数

     ```javascript
     function arrayPush(arr,...newData){}//从第二个参数开始，后面的数据全部赋值给newData参数 newData是一个数组，剩余参数只能写在形参的最后面
     ```

  3. 箭头函数

     注意事项：

     * 内部this对象指向创建期上下文对象，在函数创建时就绑定好了。普通函数的this指向是在函数的执行期绑定的
     * 不能作为构造函数，不能new
     * 没有arguements
     * 不能作为生成器函数

* Array

  ```javascript
  console.log(['a','b','c'].includes('c'))
  
  //every  如果都为真，返回真，否则为假  做全选操作
  var arr4 = [1,5,8,19]
  var res = arr4.every(v=>{return v>5})
  console.log(res)   //false
  
  //some  只要有一个为真，就为真
  var arr4 = [1,5,8,19]
  var res = arr4.some(v=>{return v>5})
  console.log(res) //true
  
  //filter
  var arr4 = [1,5,8,19]
  var res = arr4.filter(v=>{return v>5})
  console.log(res)  //[8,19]
  
  //map  映射
  var arr4 = [1,5,8,19]
  var res = arr4.map(v=>{return v*5})
  console.log(res) //[5, 25, 40, 95]
  
  //reduce 拆分  购物车计算总和
  var arr4 = [1,5,8,19]
  var res = arr4.reduce((prev,current)=>{return prev+current},0)
  console.log(res) //33 
  ```

* Object

  ```javascript
  //只改变第一个参数，如果有相同的会覆盖第一个参数的值；浅拷贝
  let obj1 = Object.assign({},{x:1,y:12})
  console.log(obj1)
  
  //定义属性  configurable/enumerable/writable 默认都为false
  obj4 = {x:1}
  obj4.y = 12;
  Object.defineProperty(obj4,'z',{value:88})
  console.log(obj4)
  ```

* 集合

  1. weakMap

     类似map,但是key必须是对象，特点是key是弱引用的，GC机制不考虑回收问题。

* 异步

  1. promise

     ```javascript
     //Promise 构造函数，接受一个参数 callback,把要执行的异步任务放置在callback中
     //promise内部会维护一个状态  默认是pending  成功：resolved   失败：rejected
     let p = new Promise((resolve,reject)=>{
         //当promise被实例化的时候，callback就会执行
         setTimeout(() => {
             //可以通过传入的resolve,reject两个函数，去改变当前Promise任务的状态
             //调用resolve函数，把状态改成resolved.调用reject函数，把状态改成rejected
             console.log("1");
     
             //promise对象下有一个方法：then,该方法在promise状态发生改变时，触发then的回调
     
             reject()
     
         }, 1000);
     });
     //then会接受两个参数：这两个参数都是回调，当对应的promise对象的状态变成resolved,那么then的
     //第一个callback就会被执行，如果状态变成了rejected,那么then的第二个callback就会被执行。
     // let p2 =  p.then(()=>{
     //     console.log("成功")
     // },()=>{
     //     console.log("失败")
     // });
     //then执行后，无论执行的那个回调函数，都会返回一个新的成功状态的promise对象
     p.then(()=>{
         console.log("2")
     },()=>{
         console.log("a")
         // return new Promise((resolve,reject)=>{
         //     reject()
         // })
         //上面的简写
         return Promise.reject();
     }).then(()=>{
         console.log("3")
     },()=>{
         console.log("b")
     }).then(()=>{
         console.log("4")
     }).catch(err=>{
         //catch方法和then一样，也会返回一个resolved状态的promise对象
         console.log('错了')
     }).then(()=>{
         console.log("5")
     })
     
     let ps1 = new Promise((resolve,reject)=>{
         setTimeout(() => {
             //传递参数给then
             console.log("1")
             //reject('出错了')
             resolve(100)
         }, 1000);
     }).then(data=>{
         console.log(data)
     },err=>{
         console.log(err)
     });
     
     let ps2 = new Promise((resolve,reject)=>{
         setTimeout(() => {
             //传递参数给then
             console.log("1")
             //reject('出错了')
             resolve(100)
         }, 1000);
     }).then(data=>{
         console.log(data)
         return Promise.resolve(300)
     },err=>{
         console.log(err)
     });
     
     //[]中放promise对象，都完成后执行then
     Promise.all([ps1,ps2])
         .then(data=>{
         console.log(data)//返回的data为所有执行结果的数组
     },err=>{
         console.log(err)
     });
     
     //谁先跑完触发then,另一个不管了
     Promise.race([])
     
     ```

  2. generator

     ```javascript
     function* fn(){
         console.log("1")
         yield getData();
         console.log("3");
     }
     
     function getData(){
         setTimeout(() => {
             console.log(2)
             f.next()
         }, 1000);
     }
     //返回一个迭代器函数
     let f = fn();
     f.next();
     ```

  3. async、await

     ```javascript
     function getData(){
         return new Promise((resolve,reject)=>{
             //resolve(100)
             reject('err')
         })
     }
     async function fn1(){
         console.log(222)
         try{
             let v = await getData();
             console.log(v);
             console.log(333)
         }catch(e){
             console.log(e)
         }
     }
     fn1();
     ```

     


​     

​     

​     

  

  

  

  

  

  





  

  

  

  

  