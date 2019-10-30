---
title: effective-java读书笔记2
date: 2018-07-14 17:46:55
tags: java
---

##### 第一条：考虑用静态工厂方法代替构造器

###### 优势：

- 静态工厂方法有名字，当一个类需要多个带有相同签名的构造器时，就用静态工厂方法代替构造器，并且慎重的选择名称以便突出他们之间的区别。

- 不必在每次调用它们的时候都创建一个新对象。

- 它们可以返回原返回类型的任何子类型的对象。这样我们在选择返回对象的类时就有了更大的灵活性。

- 创建参数化类型实例的时候，它们使代码变得更加简洁(但在新版本的java中已经可以省略)

  ```java
  List<String> list = new ArrayList<String>();
  ```
  
  <!--more-->

劣势

- 类如果不含公有的或者受保护的构造器，就不能被子类化。
- 它们与其它的静态方法实际上没有任何区别。在API文档中，它们没有像构造器那样在API文档中明确的标识出来。因此要想查明如何实例化一个类非常困难。

##### 第六条：避免创建不必要的对象

```java
String s = new String("bikini"); // DON'T DO THIS!
String s = "bikini"; //good
```

判断一个字符串是否是一个合法的罗马数字：

```java
// Performance can be greatly improved! 每次调用都会创建Pattern实例，非常昂贵的
static boolean isRomanNumeral(String s) {
return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}

// Reusing expensive object for improved performance
public class RomanNumerals {
	private static final Pattern ROMAN = Pattern.compile(
        "^(?=.)M*(C[MD]|D?C{0,3})"
        + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
	static boolean isRomanNumeral(String s) {
		return ROMAN.matcher(s).matches();
	}
}
```

自动装箱、拆箱（要优先使用基本类型而不是装箱基本类型，当心无意识的自动装箱）

```java
private static long sum() {
    Long sum = 0L; //使用long将更快
	for (long i = 0; i <= Integer.MAX_VALUE; i++)
		sum += i;
	return sum;
}
```

##### 第七条：消除过期的对象引用

```java
// Can you spot the "memory leak"?
public class Stack {
	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
    public Stack() {
    	elements = new Object[DEFAULT_INITIAL_CAPACITY];
    } 
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    } 
    public Object pop() {
        if (size == 0)
        	throw new EmptyStackException();
        return elements[--size];
    } /
    **
    * Ensure space for at least one more element, roughly
    * doubling the capacity each time the array needs to grow.
    */
    private void ensureCapacity() {
        if (elements.length == size)
        	elements = Arrays.copyOf(elements, 2 * size + 1);
}}
```

解决内存泄漏

```java
public Object pop() {
    if (size == 0)
    	throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```

**一般而言，只要类时自己管理内存，程序员就应该警惕内存泄漏问题**，一旦元素被释放掉，则该元素中包含的任何对象引用都应该被清空。

##### 第八条：覆盖equals时请遵守通用约定。

- 什么条件下，不需要覆盖equals:

  > 类的每个实例本质上都是唯一的。代表活动实体而不是值的类来说确实如此。
  >
  > 不关心类是否提供了“逻辑相等“的测试功能。
  >
  > 超类已经覆盖了equals,从超类继承过来的行为对于子类也是合适的。
  >
  > 类是私有的或者包级私有的，可以确定它的equals方法永远不会被调用。

- 什么时候应该覆盖equals方法：

  > 如果类具有自己特有的“逻辑相等”概念（不同于对象等同的概念），而且超类还没有覆盖equals以实现期望的行为。

- 覆盖时必须遵守的通用约定：

  > 自反性：x.equals(x)返回true。
  >
  > 对称性：x.equals(y) == y.equals(x)。
  >
  > 传递性：x.equals(y)  y.equals(z)  x.equals(z)。
  >
  > 一致性：只要对象没有被修改，多次调用返回一致。

  **我们无法在扩展	可实例化的类的同时，既增加新的值组件，同时又保留equals约定**

- 实现高质量equals方法的诀窍：

  1. 使用==操作符检查“参数是否为这个对象的引用”。
  2. 使用instanceof操作符检查参数是否为正确的类型。
  3. 把参数转换成正确的类型。
  4. 检查参数中的域是否与该对象中对应的域相匹配。对于既不是float也不是double类型的基本类型域，可以使用==操作符进行比较；对于对象引用域，可以递归的调用equals方法；对于float域，可以使用Float.compare方法，对于double域，则使用Double.compare。对于数组域，Arrays.equals()方法。

