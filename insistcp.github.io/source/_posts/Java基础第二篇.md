---
title: Java基础第二篇
date: 2017-03-27 22:20:17
tags: [Java]
---
#### 问题一：Java的三大特性
##### 1：封装
封装又称为信息的隐藏，是指利用抽象数据类型将数据和对数据的操作封装到一起的，使其构成一个不可分割的独立实体，数据被保护在抽象数据类性的内部，尽可能的隐藏内部的细节。只保留一些对外的接口。通过这些接口和外界发生联系。
优点：
+ a：实现了专业的分工。
	将能够实现某一特定功能的代码封装成一个独立的实体，各个程序在需要使用时直接使用它对外开放的接口进行调用即可。
+ b：隐藏了信息的实现细节。
	通过访问权限控制可以将不想让用户看到的信息隐藏起来。
<!--more-->

##### 2：继承
概念：继承就好比父子关系，继承的类称为子类，被继承的类称为父类
目的：实现代码的复用Java语言以单继承多实现的来减少代码的冗余。
理解：子类和父类是一种特殊化和一般化的关系，子类是更加详细的父类
结果：继承之后子类就拥有了父类的属性和方法，PS：私有属性和构造方法是不能继承的。补充一点：子类可以添加新的方法也可以重写父类的方法，但是对于静态方法子类不能重写父类的。
```java
	public class Son extends father{
		public void walk(){
			System.out.println("我是子类的walk方法");
		}
		public static void say(){
			System.out.println("我是子类的静态say方法");
		}
		public static void main(String[] args) {
			Son son = new Son();
			father fson = new Son();
			son.say();
			son.walk();
			son.eat();
			fson.walk();
			fson.say();
		}
	}
	class father{
		private char sex;
		public int age;
		public static void say(){
			System.out.println("我是父类的静态say方法");
		}
		public static void eat(){
			System.out.println("我是父类的静态eat方法");
		}
		public void walk(){
			System.out.println("我是父类的普通walk方法");
		}
	}
	```
由以上测试我们可知，对于普通方法而言，子类会统统从父类哪里继承过来，但是对于静态方法而言，虽然也会继承但是并没被重写重写而是被隐藏。只要父类的引用它就还是调用父类的静态方法。子类的引用时如果重写了就调用子类的静态方法，否则调用父类继承来的静态方法。但是记住一般情况下静态方法我们都使用类.方法名的方式对其进行访问。

##### 3：多态
Java的多态主要由两种体现形式：
+ A：通过继承体现的，就像我们上面讲的，多个子类继承一个父类，并重写了父类的同一个方法，这是我们可以通过一个父类的引用指向我们子类的对象，而调用时，根据我们具体的子类对象调用相应的方法，前提是这些调用的方法是普通方法，静态方法不可以原因上面已经讲过了。
+ B：通过在同一个类种定义多个同名方法，但是他们的参数不同，调用这些方法是根据具体参数调用不同的方法。

#### 问题二：重写和重载
+ 重写：发生在继承关系中，子类重写父类的方法，要求方法名称，参数列表返回值类型必须相同，并且重写的方法不能够使用比父类更加严格的访问权限。
+ 重载：在同一类中，方法名称相同，参数不同的方法之间构成重载（返回值无所谓）

#### 问题三：类的访问权限：
+ 1：public：所有类都可访问
+ 2: protected：同一类，或者同一包，或者不同包但是是其子类可以访问
+ 3: default: 同一类，同一个包中可以访问。
+ 4：private 仅仅在同一个类中才可以访问

#### 问题四：object的各种方法：
```java 
1： public final native Class<?> getClass();
2： public native int hashCode();
3： public boolean equals(Object obj) {
        	return (this == obj);
    	  }
4：protected native Object clone() throws CloneNotSupportedException;
5：public String toString() {
        	return getClass().getName() + "@" + Integer.toHexString(hashCode());
    	}
6：public final native void notify();
7：public final native void notifyAll();
8：public final native void wait(long timeout) throws InterruptedException;
9：public final void wait() throws InterruptedException {
        	wait(0);
    	 }
10：protected void finalize() throws Throwable { }
```