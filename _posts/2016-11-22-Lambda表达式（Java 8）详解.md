---
date: 2016-11-22  UTC
title: Lambda表达式（Java）详解
description: 在java 8 之前，我们通过匿名内部类实现带方法的接口，并且复写接口方法，代码显得很臃肿。使用了lambda之后，代码简洁了很多，对于熟悉lambda的，代码也能更容易理解和阅读。
permalink: /posts/lambda/
key: 100011
labels: [Java基础]
encoding: UTF-8
---

上篇文章讲了关于java 8 新特性的default方法的介绍，它的诞生一部分也是为了给Lambda表达式提供便利。

在java 8 之前，我们通过匿名内部类实现带方法的接口，并且复写接口方法，代码显得很臃肿。比如常见的有Runnable，Comparator等：

```
	 Runnable runnable = new Runnable() {
	     @Override
	     public void run() {
			 System.out.println("123");
	     }
	 };
```

对于只有一个方法的接口，在Java 8中，现在可以把它视为一个函数，用lambda表达式简化如下：

```
Runnable runnable1 = ()-> System.out.println("123");
```
使用了lambda之后，代码简洁了很多，对于熟悉lambda的，代码也能更容易理解和阅读。

> **基本语法**

lambda表达式支持Java也能进行简单的“函数式编程”。 可以理解为一个匿名内部类，但是又和匿名内部类有所不同，只能有一个显示声明的抽象方法。

这里引入了一个新的概念，**函数接口**（**Functional Interface**）：任何接口，如果只包含 唯一 一个抽象方法，那么它就是一个**函数接口**。为了让编译器帮助我们确保一个接口满足FI的要求（也就是说有且仅有一个抽象方法），Java8提供了@FunctionalInterface注解，用于编译级错误检查，表示必须符合函数式接口。（之前它们被称为SAM类型，即单抽象方法类型（Single Abstract Method））
例如：上面提到的Runnable，Comparator接口在 jdk1.8 之后，也改成了函数接口：

```
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

jdk在java.util.function包中预定了很多函数接口，例如：Function、Consumer、Predicate等，在Stream中比较常用到。

Lambda表达式一般的语法是：

```
	（parameters） -> expression
	（parameters） -> {statements；}
	也就是例如
	(int x, int y) -> x + y
	() -> 42
	(String s) -> { System.out.println(s); }

```

真正开发中代码的例子：

```
	//自定义的函数接口
	@FunctionalInterface
	public interface A {
	    void method1();
	}

	//用lambda表达式进行调用
	public class Test {
	    public static void main(String[] args) {
	        A a = () -> System.out.println("run A.method1");
	        a.method1();
	    }
	}

```

> **方法引用**

有时候Lambda表达式的代码就只是一个简单的方法调用而已，遇到这种情况，Lambda表达式还可以进一步简化为 方法引用（Method References） 。一共有四种形式的方法引用 ： 静态方法引用、对象的实例方法引用、类的实例方法引用、构造函数引用。

```
// 静态方法引用
List<Integer> ints = Arrays.asList(1, 2, 3);
ints.sort(Integer::compare);

//对象的实例方法引用
List<String> strings = Arrays.asList("1", "2", "3");
strings.forEach(System.out::println);

//类的实例方法引用
strings.stream().map(word -> word.length()); // lambda
strings.stream().map(String::length); // method reference

//构造函数引用
strings.stream().map(StringBuilder::new);

```

使用引用确实让代码简洁到极致，但是很容易让人看不懂，使用时需要考虑下维护人员的心情。
在上面我们多次出现的stream()方法，这个也是jdk 1.8 之后新增的，基本在所有的集合类接口中都加了这个default方法，有需要的可以自己去了解下。

> **总结**

在现代编程中，函数式编程慢慢成为流行，lambda表达式就是一种函数式编程的语法糖，而且很多集合提供了stream（）方法，使用lambda来进行操作，能充分发挥多cpu的潜能。

