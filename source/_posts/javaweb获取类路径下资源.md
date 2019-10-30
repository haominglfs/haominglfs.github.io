---
title: javaweb获取类路径下资源
date: 2019-01-24 21:30:22
tags: java
---

1. 项目结构：

   ![](https://raw.githubusercontent.com/haominglfs/images/master/20190124213320.png)

   <!--more-->

2. 类路径：指的是编译后的class文件的位置，如果是IDEA的话，一般在**\项目名\out\artifacts\项目扩展名WEB-INF\classes\a.txt** 

3. 编译后文件路径

   ![](https://raw.githubusercontent.com/haominglfs/images/master/20190124213734.png)

4. 获取类路径下资源的方式

   *  ClassLoader

     ```java
     @Controller
     public class PathController {
         @RequestMapping("/path")
         public void testPath() throws IOException {
             //如果资源文件不是直接在src下，而是在其他包下面,要改成			getResourceAsStream("com/haominglfs/test/a.txt")  开头没有斜杠 
             InputStream resourceAsStream =
                     this.getClass().getClassLoader().getResourceAsStream("a.txt");
             BufferedReader br = new BufferedReader(new InputStreamReader(resourceAsStream,"UTF-8"));
             String s = br.readLine();
             System.out.println(s);
         }
     }
     ```

   * Class 

     ```
     //得到Class
     Class c = this.getClass();
     //相对于当前.class文件所在目录,开头还是没有斜杆的
     InputStream input = c.getResourceAsStream("a.txt");
     ```

     如果资源文件和.class文件不同目录

     ```java
     getResourceAsStream("/a.txt");   注意：这里加了一个斜杆
     ```

     