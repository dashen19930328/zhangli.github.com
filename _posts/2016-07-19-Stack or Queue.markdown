---
layout:     post
title:      "Java Queue与Stack的使用"
subtitle:   "Java Queue与Stack的使用"
date:       2016-07-19
author:     "Mr.baicai"
header-img: "img/header-post/2016-03-20-01.jpg"
tags:
    - java研究
---

> 版权声明：本文为博主原创文章，未经博主允许不得转载。

#   1.Queue

##  1.1  queue

使用频率高的数据结构之一，其在jdk中的源码如下。

```
  public interface Queue<E> extends Collection<E> {
    boolean add(E e);
    boolean offer(E e);
    E remove();
    E  poll();
    E element();
    E peek();}
      
```
    
##   1.2 由源码知

 queue是一个FIFO的队列数据结构，可用于生活中很多场景，比如吃饭排队等等，其继承Collection接口， 本身也是接口。
 
##  1.3 queue中方法
    * add(),在队列的末尾添加元素，如果队列满，则抛出IllegalStateException 的异常
    * offer(),向队列尾添加元素，如果队满，则返回false
    * remove,删除队列头的元素，若队列为空，则抛出NoSuchElementException异常
    * poll(),返回并删除队列头的元素，若队列为空，则返回null
    * element(),返回队列头的元素，若队列为空，则返回NoSuchElementException异常
     8 peek(),返回队列头的元素，若队列为空，则返回null
                          
#  2.Stack

##   2.1 stack

也是编程中比较常使用到的数据结构之一，jdk源码如下。
     
```
  public class Stack<E> extends Vector<E> {
    public Stack() {
    }
    public E push(E item) {
        addElement(item);
        return item;
    }
    public synchronized E pop() {
        E   obj;
        int len = size();
        obj = peek();
        removeElementAt(len - 1);
        return obj;
    }
    public synchronized E peek() {
        int len = size();
        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }
    public boolean empty() {
        return size() == 0;
    }
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);
        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }
}
```

##  2.2 stack

  继承vector的一个抽象类，是一种FILO的数据结构，应用场景，个人理解为queue受限的情景下，当然从源码来看，stack完全可以通过queue来模拟使用。

##  2.3 stack中方法

*   push(),向栈顶添加元素，并返回该元素
*   pop(),从栈顶删除元素，并返回该元素，若栈为空，则抛出EmptyStackException异常，其中使用到 synchronized ，来保证多线程编程下，数据操作的一致性
*    peek（），返回栈顶元素，返回该元素，若栈为空，则抛出EmptyStackException异常
*    empty（），通过栈中元素个数来判断栈是否为空等
*    search(),寻找并返回从栈顶1开始计数的一个累加值，即元素位置，若不存在则返回-1
    
##  2.4 最后说明

  若要熟悉或者深入理解机理用法，需要查看jdk源码与自己实践操作,queue与stack是可以互相模拟来使用。                          
