# ConcurrentLinkedQueue
## 概述
我们要实现一个线程安全的队列有两种实现方法一种是使用阻塞算法，另一种是使用非阻塞算法。使用阻塞算法的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现，而非阻塞的实现方式则可以使用循环CAS的方式来实现，我们就来研究下ConcurrentLinkedQueue是如何保证线程安全的同时又能高效的操作的。
## ConcurrentLinkedQueue的结构
ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，当我们获取一个元素时，它会返回队列头部的元素。基于CAS的“wait－free”（常规无等待）来实现，CAS并不是一个算法，它是一个CPU直接支持的硬件指令，这也就在一定程度上决定了它的平台相关性。
当前常用的多线程同步机制可以分为下面三种类型：
- volatile 变量：轻量级多线程同步机制，不会引起上下文切换和线程调度。仅提供内存可见性保证，不提供原子性。
- CAS 原子指令：轻量级多线程同步机制，不会引起上下文切换和线程调度。它同时提供内存可见性和原子化更新保证。
- 互斥锁：重量级多线程同步机制，可能会引起上下文切换和线程调度，它同时提供内存可见性和原子性。
ConcurrentLinkedQueue 的非阻塞算法实现主要可概括为下面几点：
- 使用 CAS 原子指令来处理对数据的并发访问，这是非阻塞算法得以实现的基础。
- head/tail 并非总是指向队列的头 / 尾节点，也就是说允许队列处于不一致状态。 这个特性把入队 /出队时，原本需要一起原子化执行的两个步骤分离开来，从而缩小了入队 /出队时需要原子化更新值的范围到唯一变量。这是非阻塞算法得以实现的关键。
- 以批处理方式来更新head/tail，从整体上减少入队 / 出队操作的开销。

ConcurrentLinkedQueue由head节点和tail节点组成，每个节点（Node）由节点元素（item）和指向下一个节点的引用(next)组成，节点与节点之间就是通过这个next关联起来，从而组成一张链表结构的队列。默认情况下head节点存储的元素为空，tail节点等于head节点。
## 入队列
入队主要做两件事情，第一是将入队节点设置成当前队列尾节点的下一个节点。第二是更新tail节点，在入队列前如果tail节点的next节点不为空，则将入队节点设置成tail节点，如果tail节点的next节点为空，则将入队节点设置成tail的next节点，所以tail节点不总是尾节点。
上面的分析从单线程入队的角度来理解入队过程，但是多个线程同时进行入队情况就变得更加复杂，因为可能会出现其他线程插队的情况。如果有一个线程正在入队，那么它必须先获取尾节点，然后设置尾节点的下一个节点为入队节点，但这时可能有另外一个线程插队了，那么队列的尾节点就会发生变化，这时当前线程要暂停入队操作，然后重新获取尾节点。让我们再通过源码来详细分析下它是如何使用CAS方式来入队的(JDK1.8)：
```java
public boolean offer(E e) {
        checkNotNull(e);
        //创建入队节点
        final Node<E> newNode = new Node<E>(e);
        //t为tail节点，p为尾节点，默认相等，采用失败即重试的方式，直到入队成功
        for (Node<E> t = tail, p = t;;) {
            //获得p的下一个节点
            Node<E> q = p.next;
            // 如果下一个节点是null,也就是p节点就是尾节点
            if (q == null) {
              //将入队节点newNode设置为当前队列尾节点p的next节点
                if (p.casNext(null, newNode)) { 
                   //判断tail节点是不是尾节点，也可以理解为如果插入结点后tail节点和p节点距离达到两个结点
                    if (p != t) 
                     //如果tail不是尾节点则将入队节点设置为tail。
                     // 如果失败了，那么说明有其他线程已经把tail移动过 
                        casTail(t, newNode);  
                    return true;
                }
            }
                 // 如果p节点等于p的next节点，则说明p节点和q节点都为空，表示队列刚初始化，所以返回                            head节点
            else if (p == q)
                p = (t != (t = tail)) ? t : head;
            else
                //p有next节点，表示p的next节点是尾节点，则需要重新更新p后将它指向next节点
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }
```
从源代码我们看出入队过程中主要做了三件事情，第一是定位出尾节点；第二个是使用CAS指令将入队节点设置成尾节点的next节点，如果不成功则重试；第三是重新定位tail节点。
从第一个if判断就来判定p有没有next节点如果没有则p是尾节点则将入队节点设置为p的next节点，同时如果tail节点不是尾节点则将入队节点设置为tail节点。如果p有next节点则p的next节点是尾节点，需要重新更新p后将它指向next节点。还有一种情况p等于p的next节点说明p节点和p的next节点都为空，表示这个队列刚初始化，正准备添加数据，所以需要返回head节点。
每次出队时都更新head节点，当head节点里有元素时，直接弹出head节点里的元素，而不会更新head节点。只有当head节点里没有元素时，出队操作才会更新head节点。让我们再通过源码来深入分析下出队过程(JDK1.8):
```java
 public E poll() {
        // 设置起始点  
        restartFromHead:
        for (;;) {
        //p表示head结点，需要出队的节点
            for (Node<E> h = head, p = h, q;;) {
            //获取p节点的元素
                E item = p.item;
                //如果p节点的元素不为空，使用CAS设置p节点引用的元素为null
                if (item != null && p.casItem(item, null)) {
                    
                    if (p != h) // hop two nodes at a time
                    //如果p节点不是head节点则更新head节点，也可以理解为删除该结点后检查head是否与头结点相差两个结点，如果是则更新head节点
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    return item;
                }
                //如果p节点的下一个节点为null，则说明这个队列为空，更新head结点
                else if ((q = p.next) == null) {
                    updateHead(h, p);
                    return null;
                }
                //结点出队失败，重新跳到restartFromHead来进行出队
                else if (p == q)
                    continue restartFromHead;
                else
                    p = q;
            }
        }
    }
```
更新head节点的updateHead方法：
```java
final void updateHead(Node<E> h, Node<E> p) 
{
     // 如果两个结点不相同，尝试用CAS指令原子更新head指向新头节点
     if (h != p && casHead(h, p))
         //将旧的头结点指向自身以实现删除
     h.lazySetNext(h);
}
```
首先获取head节点的元素，并判断head节点元素是否为空，如果为空，表示另外一个线程已经进行了一次出队操作将该节点的元素取走，如果不为空，则使用CAS的方式将head节点的引用设置成null，如果CAS成功，则直接返回head节点的元素，如果CAS不成功，表示另外一个线程已经进行了一次出队操作更新了head节点，导致元素发生了变化，需要重新获取head节点。如果p节点的下一个节点为null，则说明这个队列为空（此时队列没有元素，只有一个伪结点p），则更新head节点。
## 队列判空
有些人在判断队列是否为空时喜欢用queue.size()==0，让我们来看看size方法：
```java
public int size() 
{
     int count = 0;
     for (Node<E> p = first(); p != null; p = succ(p))
         if (p.item != null)
             // Collection.size() spec says to max out
             if (++count == Integer.MAX_VALUE)
                 break;
     return count;
 }
```
可以看到这样在队列在结点较多时会依次遍历所有结点，这样的性能会有较大影响，因而可以考虑empty函数，它只要判断第一个结点(注意不一定是head指向的结点)。
```java
public boolean isEmpty() {
        return first() == null;
    }
```
