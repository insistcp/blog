---
title: Java基础第一篇
date: 2017-03-26 13:15:11
tags: [Java]
---
#### 引言：最近知乎上看了姚晟同学实习经历以及它的知识储备，深感惭愧，同样是Java开发，他的学习以及理解让自己有一种不可企及的感觉，遂决定从头开始完完整整的将所有的基础全补习一遍，按照（姚晟）推荐的顺序：
#### 一：Java基础之-Java的内存模型
+ 1.1:内存模型概述
正如我们所知道的那样，我们电脑的CPU执行速度十分快，就算是读写速度十分快的内存和CPU的计算速度相比也差了很多量级；所以为了让Java应用程序充分利用CPU,Java虚拟机专门设定了自己的内存模型。如下图所示：
<center>![Java虚拟机的内存模型](/Java基础第一篇/1.jpg)</center>

<!--more-->
主内存就是我们电脑分配给Java虚拟机的内存（一般都是指所有线程共享的内存段）为了让每一个应用程序能够不用等待从内存中排队拿数据，我们在主内存和应用程序之间增加了工作内存（每一个线程私有的）。如果线程A想要使用内存中的一个变量，那么我们的Java虚拟机会把线程A想要使用的数据copy到线程A的工作内存中，然后线程A先在自己的工作内存中对数据进行操作，操作完成之后，再将数据刷回内存中。如果仅仅考虑这三成，大家想想有没有什么问题？如果每一线程对内存中同一数据都做了更改，例如某一刻内存中的变量A=3；线程A想要对这个变量进行一个+10操作，现在线程A从从内存中读取A将其Copy到自己的工作内存中之后进行相关操作，此时线程B也相对A进行操作--对变量A进行-10操作，一样，先到内存中奖变量取出来（copy一份到自己的工作内存）再在工作内存中对其进行相关操作，此时如果线程A先将自己的结果刷到内存中，那么线程B将会覆盖线程A的计算结果，如果线程B先将计算结果刷进内存，线程A将会覆盖线程B的计算结果。当然还有一种可能是线程A执行完了线程B再来取值（正常流程）同样的操作，如果取值时机不同将会得到截然不同的结果。所以如果仅仅使用如上结构而不加任何限制的话，明显是不可行的。所以我们在工作内存和主内存之间进行数据传输时，需要添加一定的限制（Load和Save操作）保证执行结果的准确性。
关于主内存与工作内存之间的交互协议，即一个变量如何从主内存拷贝到工作内存。如何从工作内存同步到主内存中的实现细节。java内存模型定义了8种操作来完成。这8种操作每一种都是原子操作。8种操作如下：
 + lock(锁定)：作用于主内存，它把一个变量标记为一条线程独占状态；
 + unlock(解锁)：作用于主内存，它将一个处于锁定状态的变量释放出来，释放后的变量才能够被其他线程锁定；
 + read(读取)：作用于主内存，它把变量值从主内存传送到线程的工作内存中，以便随后的load动作使用；
 + load(载入)：作用于工作内存，它把read操作的值放入工作内存中的变量副本中；
 + use(使用)：作用于工作内存，它把工作内存中的值传递给执行引擎，每当虚拟机遇到一个需要使用这个变量的指令时候，将会执行这个动作；
 + assign(赋值)：作用于工作内存，它把从执行引擎获取的值赋值给工作内存中的变量，每当虚拟机遇到一个给变量赋值的指令时候，执行该操作；
 + store(存储)：作用于工作内存，它把工作内存中的一个变量传送给主内存中，以备随后的write操作使用；
 + write(写入)：作用于主内存，它把store传送值放到主内存中的变量中。
Java内存模型还规定了执行上述8种基本操作时必须满足如下规则:

 + 不允许read和load、store和write操作之一单独出现，以上两个操作必须按顺序执行，但没有保证必须连续执行，也就是说，read与load之间、store与write之间是可插入其他指令的。
 + 不允许一个线程丢弃它的最近的assign操作，即变量在工作内存中改变了之后必须把该变化同步回主内存。
 + 不允许一个线程无原因地（没有发生过任何assign操作）把数据从线程的工作内存同步回主内存中。
一个新的变量只能从主内存中“诞生”，不允许在工作内存中直接使用一个未被初始化（load或assign）的变量，换句话说就是对一个变量实施use和store操作之前，必须先执行过了assign和load操作。
 + 一个变量在同一个时刻只允许一条线程对其执行lock操作，但lock操作可以被同一个条线程重复执行多次，多次执行lock后，只有执行相同次数的unlock操作，变量才会被解锁。
 + 如果对一个变量执行lock操作，将会清空工作内存中此变量的值，在执行引擎使用这个变量前，需要重新执行load或assign操作初始化变量的值。
 + 如果一个变量实现没有被lock操作锁定，则不允许对它执行unlock操作，也不允许去unlock一个被其他线程锁定的变量。
对一个变量执行unlock操作之前，必须先把此变量同步回主内存（执行store和write操作）。

+ 1.2：重排序
Java虚拟机在我们的程序在执行过程中为了提高程序的并行度，编译器和处理器通常会对我们的程序指令重新排序（前提是不改变程序执行结果）：
通常有以下三种重排序：
 * 1：编译器优化重排序
 	是指我们编译器在将我们的源代码编译成class文件的过程中会重新安排语句执行的顺序（前提是部影响单线程下程序的语义）
 * 2：指令级并行重排序
 	主要是指我们的处理器采用了指令并行技术允许多条指令重叠执行。（前提是不存在数据依赖）
 * 3：内存系统的重排序
	因为虚拟机使用了缓冲区去读写缓冲区，使得我们的代码开上去好像是乱序执行的。
例子一：单线程下重排序
```java
	double PI = 3.14; //1
	double r = 10; //2
	double square = PI*r*r; //3
```
	看上面的代码：第3步的计算依赖于第1步和第2步的结果，但是步骤1和步骤2之间并不存在以来关系，所以程序在执行过程中，1和2之间可以进行重排序，但是步骤3必须在1和2之后，3不能和1，2重排序；
例子二：多线程下的重排序：
```
class ReorderExample {
	int a = 0;
	boolean flag = false;
	public void writer() {
	    a = 1;                   //1
	    flag = true;             //2
	}
	public void reader() {
	    if (flag) {                //3
	        int i =  a * a;        //4
	        ……
	    }
	}
}
```
	看上面的代码，现在我们有两个线程分别执行上述代码，假设线程A先执行写操作，线程B后执行读操作；步骤1和2之间没有数据依赖，按上面的原则他们之间是可以进行重排序的：
	![Java虚拟机的内存模型](/Java基础第一篇/1.png)
	假设我们的现在对指令1，2进行重排序，程序执行过程如上图所示，此时程序执行的最终结果和预料的完全不同，语义已经被破坏。
	同理我们再来看指令3，4他们之间也没有相互依赖关系，我们对其进行重排序：程序执行流程如下图所示：
	![Java虚拟机的内存模型](/Java基础第一篇/2.png)
	语义又一次被破坏了；
所以由上面两种情况我们可知，重排序对于单线程的程序而言，如果指令之间无依赖，没问题。但是对于多线程的程序而言，就算指令之间没有依赖，重排序也有可能会破环我们原来的语义。
 + 1.3：同步机制：
 	Java虚拟机中保证各个线程工作内存中的数据同步主要有以下三种方式：
 	* 1：volatile:
 	A：如果一个变量使用volatile关键字修饰，那么我们对该变量的操作将具有原子性和可见性.提示volatile修饰的变量对v++这种操作也不具有原子性。
 	PS:原子性：执行过程不可中断，要么执行成功要么失败。
 	可见性：对一个变量的读总能看到其他线程对它的写。
	B:volatile内存语义：
		volatile写：当一个变量被volatile修饰时线程对其的写操作会直接刷到主存中
		volatile读：JMM会将本地内存的改变量设为无效而直接从内存中读取改变量的值。
	C:volatile和synchronize对比
		功能上监听器锁比volatile更加强大，但是从可伸缩行和执行性能上来讲，volatile更具优势。
		volatile仅仅保证单个volatile变量的读写具有原子性，而synchronized的互斥执行特性可以确保对整个临界区代码的执行具有原子性。

	* 2: synchronized：
		该关键字一般用来修饰方法或者代码块，表示给相应的对象加锁；
		A：当多个线程访问一个对象的synchronized方法时，先拿到对象锁的方法先执行其他线程依次等待，直到拿到对象锁的方法；其他线程还可以正常访问该对象的非synchronized方法。
		B：当多个线程访问多个对象的普通方法，正常访问。
		C：synchronized锁重入：
		指当一个线程得到该对象的锁之后，再次请求该对象锁时可以直接得到该对象的锁。
		```java
		public class demo {
			public synchronized void methodA(){
				System.out.println("methodA is running"+Thread.currentThread().getName());
				methodB();
			}
			public synchronized void methodB(){
				System.out.println("methodB is running"+Thread.currentThread().getName());
				methodC();
			}
			public synchronized void methodC(){
				System.out.println("methodC is running"+Thread.currentThread().getName());
			}
		}
		public class ThreadTT extends Thread{
			@Override
			public void run() {
				demo de = new demo();
				de.methodA();
			}
			public static void main(String[] args) {
				ThreadTT tt = new ThreadTT();
				ThreadTT t1 = new ThreadTT();
				tt.start();
				t1.start();
			}
		}

		```
		D:sychnorized同步代码块
		其实就是缩小了同步代码的访问记得要确定我们同步锁的对象（即对象锁是什么）
		记住多个线程访问同一个对象的不同的同步方法时也是同步的。
		E：静态同步synchronized方法
		关键字synchronized还可用在静态方法上，表示对整个类进行加锁。想要同步，那么方法所持有的锁一定是一个才行。
		F：多线程死锁：
		互相持有对方所需要的锁，都不肯释放便造成死锁。
		```java
		public class deadLock extends Thread{
			private Object firstL = new Object();
			private Object secondL = new Object();
			private String username;
			public void setUsername(String username) {
				this.username = username;
			}
			@Override
			public void run() {
				if("aaa".equals(username)){
					synchronized (firstL){ 
						try{
							System.out.println("username="+username);
							Thread.sleep(10000);
								
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
						synchronized (secondL) {
							System.out.println("firstL is end");
							
						}
					}
				}
				if("bbb".equals(username)){
					synchronized (secondL){ 
						try{
							System.out.println("username="+username);
							Thread.sleep(10000);
								
						} catch (InterruptedException e) {
							// TODO Auto-generated catch block
							e.printStackTrace();
						}
						synchronized (firstL) {
							System.out.println("secondL is end");
							
						}
					}
				}
			}
			public static void main(String[] args) {
				deadLock d = new deadLock();
				d.setUsername("aaa");
				d.start();
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				deadLock d1 = new deadLock();
				d1.setUsername("bbb");
				d1.start();
			}
		}
		```
	* 3：final关键字
	对于final域，编译器和处理器要遵循两个重排序规则：
	对于基础数据类型JMM会有如下规定
	1：在构造函数内对一个final域的写入，与随后初次读这个final域这两个操作不能重排序。
	2：初次读一个包含final域的对象的引用，与随后初次读取这个final域，这两个操作之间不能重排序。
	对于引用数据类型：
	在构造函数内对一个final引用的对象成员域的写入，与随后在构造函数外把构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
	详见：[并发编程网](http://ifeve.com/java-memory-model/)