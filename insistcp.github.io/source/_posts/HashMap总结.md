---
title: HashMap总结
date: 2017-03-09 12:18:45
categories: 语言 #文章文类
tags: [Java] #文章标签，多于一项时用这种格式
---
###### 概述：尽管已经将java核心编程中的集合看了好几遍了，但是还是感觉心里没有低；所以今天特别将集合这块常见的面试题进行总结，一边加深自己对java集合的理解；
<!-- more -->，
### 一:谈谈你对HashMap具体实现的理解：
1：Hashmap是一种基于Map接口的实现,hashMap是一个键值对类型的集合；**他的键和值都可以为null;**
2：HashMap的实例有两个参数影响其性能：
    初始容量和加载因子。
    初始容量就是哈希表中的桶的数量。加载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度；
    当哈希表中的条目数超出了加载因子与当前容量的乘积时，则需要对该hash表进行rehash操作。从而让哈希表具有大约两倍的桶数。
3：HashMap的底层实现是哈希表的形式（格式像数组链表的组合）当创建一个HahsMap对象时候就会创建一个hash表；如果创建时指定了大小就按照我们指定的大小个桶的个数。（因为有加载因子的存在一般都装不满）
    <center>![HashMap](/HashMap总结/1.jpg)</center>
    **没有指定大小时默认的大小是16，默认的加载因子是0.75**
    使用hashMap存储数据时，首先我们键会先调用它的hashCode方法生成一个hashCode值，然后对这个的hashCode值再进一次移位的运算；

```java
//java中的抖动函数
static  final int hash(Object Key){
    int h;
    if(key== null){
      return null  
    }else{
     return key==null ? 0:((h=h.hashCode) ^ h>>>16)
  }
} 
```
当然这样得到的hash值是不能直接作为数组的索引存储的；太大的；改进后的hashMap初始容量才是16；所以我们还需要对这个得到的值进行一次模运算：
```java
 static int indexFor(int hash, int length) {
        return hash & (length-1);
   }
```
得到的结果便是我们想要存储的bucket的位置；此时如果篮子中没有值，我们直接将值放进这个bucket中键即可；如果有值即产生了我们所谓的hash冲突了；
散列表要解决的一个问题就是散列值的冲突问题，通常是两种方法：链表法和开放地址法。
链表法就是将相同hash值的对象组织成一个链表放在hash值对应的槽位；
开放地址法是通过一个探测算法，当某个槽位已经被占据的情况下继续查找下一个可以使用的槽位。java.util.HashMap采用的链表法的方式，链表是单向链表。形成单链表的核心代码如下：
```java
void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }
        createEntry(hash, key, value, bucketIndex);
    }
 void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
```
   
