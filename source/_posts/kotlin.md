---
title: kotlin
date: 2018-12-12 21:17:34
tags:
---

### 空类型

任意类型都有可空和不可空两种

val notNull:String = null //错误，不能为空

val nullable:String? = null //正确，可以为空

notNull.length //正确，不为空的值可以直接使用

Nullable.length //错误，可能为空，不能直接获取长度

Nullable!!.length //正确，强制认定nullable不可空

Nullable?.length //正确，若nullable为空，返回空

<!--more-->

### 智能类型转换

1. java style 类型转换

   val sub:subClass  = parent as subClass

   类似于java的类型转换，失败则抛出异常

2. 安全类型转换

   val sub :subClass = parent as? subClass

   如果转换失败，返回null,不抛出异常

### Lamdba表达式

* 写法：{[参数列表] ->  [函数体,最后一行是返回值 ]}

### var还是val?

* 原则：如果两种方式都能满足需求的情况下，优先使用val声明，因为一方面val声明的变量是只读，一旦初始化后不能修改，还可以避免程序运行时错误的修改变量的内容；另一方面在声明引用类型使用val,对象的引用不会被修改，但是引用内容可以修改，这样会更加安全，也符合函数式编程的技术要求。

### Elvis运算符

* A ?: B  如果A不为空值则结果为A,否则结果为B。



