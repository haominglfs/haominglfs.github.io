---
title: node.js
date: 2019-02-13 12:25:55
tags:
---

### node知识点

1. console

2. __dirname  当前文件所在的目录。

3. __filename  文件的全路径。

4. eventLoop

5. 宏任务、微任务

   <!--more-->

   ```javascript
   //宏任务  主体script  setTimeout  setInterval (new Promise)
   //微任务  Promise.then  process.nextTick()
   setTimeout(()=>{
       //宏任务
       new Promise(resolve =>{
           console.log("promise")
           resolve()
       }).then(()=>{
           //微任务
           console.log("then")
       })
       //宏任务
       console.log(1)
       //宏任务
       setTimeout(()=>{
           console.log(222)
       })
   })
   ```

6. 模块化（commonjs规范）

   * 一个文件就是一个独立的模块。每个模块都有自己的独立作用域—模块作用域

   * 模块加载采用同步模式。

   * 通过require导入,exports导出。module.exports == exports  返回true,但是使用上有一定的注意事项：

   * 每一个模块中都有一个内置的对象：module,该对象包括当前模块文件的一些信息
     * id   当前模块的唯一标识，默认id为当前文件的绝对路径
     * filename 当前模块的文件
     * parent
     * children
     * loaded
     * paths

   * 模块类型

     * 文件

     * 文件夹

       当我们导入的模块是一个文件夹的时候，就会引入文件夹模块导入机制：1.读取该文件夹下的package.json文件。2.导入文件中main选项指定的文件。3.如果不存在，则导入文件夹下的index.js。

       * node_modules文件夹：如果我们的模块在node_modules目录下(一般用于第三方库)

         ```javascript
         //let m2 = require('./node_modules/m2')
         let m2 = require('m2')//和上面的效果一样
         
         //如果模块的加载是以./ ../ /开头的，就是路径模块加载模式
         //不以./ ../ /开头的，按照另外一种加载机制，非路径加载模式，按照以下规则查找：
         	//在module对象有一个属性paths,里面保存的是非路径模式下的查找列表。
         paths:
            [ '/Users/haominglfs/Documents/es6/node_modules',
              '/Users/haominglfs/Documents/node_modules',
              '/Users/haominglfs/node_modules',
              '/Users/node_modules',
              '/node_modules' ]
         ```

       * global 文件夹:所有项目都可以使用的模块,在node安装目录下node_modules文件夹。

     * 核心模块：node内置模块:

       ```javascript
       let fs = require('fs')
       //如果自己定义的模块和核心模块冲突，则默认加载核心模块。
       ```

7. 模块文件后缀处理机制：

   .js>.json>.node

8. es6模块化

   ```javascript
   //导出
   export var a = 10;
   //每个模块只能存在一个default
   export default 100
   //导入
   import {a} from './m1'
   import * as m1 from './m1'
   //导入默认,直接赋值
   import a from './m1'
   
   ```

9. 内置对象

   * events

     ```javascript
     const eventEmitter = require('events')
     
     class Person extends eventEmitter{
         constructor(name){
             super()
             this.name = name;
             this.age = 0;
             this.growup();
         }
     
         growup(){
             setInterval(() => {
                 this.age++;
                 this.emit('growup') //触发事件
             }, 1000);
         }
     
     }
     
     const p1 = new Person('郝明');
     
     p1.addListener('growup',function(){ //注册监听器
         console.log('growup 1')
     })
     
     console.log(p1.eventNames())
     ```

   * process(全局对象，不需要引入)

     ```javascript
     //process.argv 属性返回一个数组，其中包含当启动 Node.js 进程时传入的命令行参数。
     console.log(process.argv)
     
     //环境变量
     console.log(process.env)
     
     //标准输入输出
     process.stdout.write('hello')
     
     process.stdin.on('data',(e)=>{
         console.log('用户输入'+e.toString())
     })
     ```

   * stream

     * 四种基本类型
       * Writable  可写入数据的流。 fs.createWriteStream()
       * Readable 可读取数据的流。 fs.createReadStream()
       * Duplex  可读又可写的流。 Net.Socket()
       * Transform 在读写过程中可以修改后转换

   * buffer

10. filesystem

    ```javascript
    const fs = require('fs')
    //First Error :node 中的一种约定，如果一个回调可能有错误发生，那么约定回调函数的第一个参数用来做//错误对象
    // fs.writeFile('./1.txt','你好我好',(err)=>{
    //     if(err) throw err
    //     console.log('文件已保存')
    // })
    
    // let res = fs.writeFileSync('./2.txt','你好','utf8')
    // console.log(res)
    
    //返回buffer
    fs.readFile('./1.txt',(err,data)=>{
        if(err) throw err
        console.log(data.toString())
    })
    
    let content = fs.readFileSync('./2.txt')
    console.log(content)
    
    try{
        let st = fs.statSync('./1.txt')
        console.log(st)
        console.log(st.isFile())
        console.log(st.isDirectory())
    }catch(e){
        console.log(e)
    }
    
    //删除
    //fs.unlinkSync('./1.txt')
    
    //文件夹
    //fs.mkdirSync('./a')
    //不能递归创建文件夹
    //fs.mkdirSync('./a/b/c')
    
    //不能删除非空文件夹
    fs.rmdirSync('./a');
    
    let dir = './a';
    // let files = fs.readdirSync(dir);
    // files.forEach(child=>{
    //     fs.unlinkSync(dir+'/'+child);
    // })
    
    rmdir(dir)
    //递归删除文件夹
    function rmdir(dirpath){
        let files = fs.readdirSync(dirpath);
        files.forEach(child=>{
            let childPath = dirpath+'/'+child;
            let st = fs.statSync(childPath)
            if(st.isDirectory()){
                rmdir(childPath)
            }else{
                fs.unlinkSync(childPath);
            }
        })
        fs.rmdirSync(dirpath)
    }
    
    //监听文件或目录
    const fs = require('fs')
    fs.watchFile('./2.txt',(e)=>{
        console.log(e)
    })
    ```

11. npm

    * 能不安装全局就不安装全局。
    * 命令行工具(第三方模块)：
      * commander
      * chalk
      * Inquirer

​       

   

   