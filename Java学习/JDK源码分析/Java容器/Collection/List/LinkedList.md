# LinkedList

### LinkedList简介

LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。
LinkedList 实现 List 接口，能对它进行队列操作。
LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
LinkedList 是非同步的。

LinkedList相对于ArrayList来说，是可以快速添加，删除元素，ArrayList添加删除元素的话需移动数组元素，可能还需要考虑到扩容数组长度。

### LinkedList属性

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
	//当前拥有多少个节点
    transient int size = 0;

    //第一个节点
    transient Node<E> first;

    //最后一个节点
    transient Node<E> last;

...
```

### LinkedList构造方法

默认构造方法是空的，什么都没做，表示初始化的时候size为0，first和last的节点都为空：

```java
public LinkedList() {
    }
```

另一个构造方法是带Collection值得对象作为入参的构造函数的，下面是执行逻辑：
1）使用this（）调用默认的无参构造函数。

2）调用addAll（）方法，传入当前的节点个数size，此时size为0，并将collection对象传递进去

3）检查index有没有数组越界的嫌疑

4）将collection转换成数组对象a

5）循环遍历a数组，然后将a数组里面的元素创建成拥有前后连接的节点，然后一个个按照顺序连起来。

6）修改当前的节点个数size的值

7）操作次数modCount自增1.

```java
public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

调用带参数的addAll方法

```java
public boolean addAll(Collection<? extends E> c) {
        return addAll(size, c);
    }
```

将collection对象转换成数组链表

```java
public boolean addAll(int index, Collection<? extends E> c) {
        checkPositionIndex(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        if (numNew == 0)
            return false;

        Node<E> pred, succ;
        if (index == size) {
            succ = null;
            pred = last;
        } else {
            succ = node(index);
            pred = succ.prev;
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;
            Node<E> newNode = new Node<>(pred, e, null);
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```

### add方法

#### add(E e)

该方法直接将新增的元素放置链表的最后面，然后链表的长度（size）加1，修改的次数（modCount）加1

```java
public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

linkLast

```java
void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }
```

#### add(int index, E element)

指定位置往数组链表中添加元素

1）检查添加的位置index 有没有小于等于当前的长度链表size，并且要求大于0

2）如果是index是等于size，那么直接往链表的最后面添加元素，相当于调用add(E e)方法

3）如果index不等于size，则先是索引到处于index位置的元素，然后在index的位置前面添加新增的元素。

```java
public void add(int index, E element) {
        checkPositionIndex(index);

        if (index == size)
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```

把索引到的元素添加到新增的元素之后

```java
void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        final Node<E> newNode = new Node<>(pred, e, succ);
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        size++;
        modCount++;
    }
```

### get方法

#### get

首先是判断索引位置有没有越界，确定完成之后开始遍历链表的元素，那么从头开始遍历还是从结尾开始遍历呢，这里其实是要索引的位置与当前链表长度的一半去做对比，如果索引位置小于当前链表长度的一半，否则从结尾开始遍历

```java
public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

遍历链表元素

```java
Node<E> node(int index) {
        // assert isElementIndex(index);

        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

#### getfirst

直接将第一个元素返回

```java
public E getFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return f.item;
    }
```

#### getlast

直接将最后一个元素返回

```java
public E getLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return l.item;
    }
```

### remove

#### remove

remove方法本质调用的还是removeFirst方法

```java
public E remove() {
        return removeFirst();
    }
```

#### removeFirst

移除第一个节点，将第一个节点置空，让下一个节点变成第一个节点，链表长度减1，修改次数加1，返回移除的第一个节点。

```java
public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }
```

unlinkFirst

```java
private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        final E element = f.item;
        final Node<E> next = f.next;
        f.item = null;
        f.next = null; // help GC
        first = next;
        if (next == null)
            last = null;
        else
            next.prev = null;
        size--;
        modCount++;
        return element;
    }
```

#### removeLast

移除最后一个节点，将最后一个节点置空，最后一个节点的上一个节点变成last节点，，链表长度减1，修改次数加1，返回移除的最后一个节点。

```java
public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }
```

#### remove(int index)

先是检查移除的位置是否在链表长度的范围内，如果不在则抛出异常，根据索引index获取需要移除的节点，将移除的节点置空，让其上一个节点和下一个节点对接起来。

```java
public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
```

```java
private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```

#### remove(Object o)

移除此列表中第一個出現的指定元素，如果它存在。如果此列表中不包含該元素，此列表是不變的。

```java
public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

```java
E unlink(Node<E> x) {
        // assert x != null;
        final E element = x.item;
        final Node<E> next = x.next;
        final Node<E> prev = x.prev;

        if (prev == null) {
            first = next;
        } else {
            prev.next = next;
            x.prev = null;
        }

        if (next == null) {
            last = prev;
        } else {
            next.prev = prev;
            x.next = null;
        }

        x.item = null;
        size--;
        modCount++;
        return element;
    }
```

### set

检查设置元素位然后置是否越界，如果没有，则索引到index位置的节点，将index位置的节点内容替换成新的内容element，同时返回旧值。

```java
public E set(int index, E element) {
        checkElementIndex(index);
        Node<E> x = node(index);
        E oldVal = x.item;
        x.item = element;
        return oldVal;
    }
```

### clear

将所有链表元素置空，然后将链表长度修改成0，修改次数加1

```java
public void clear() {
        // Clearing all of the links between nodes is "unnecessary", but:
        // - helps a generational GC if the discarded nodes inhabit
        //   more than one generation
        // - is sure to free memory even if there is a reachable Iterator
        for (Node<E> x = first; x != null; ) {
            Node<E> next = x.next;
            x.item = null;
            x.next = null;
            x.prev = null;
            x = next;
        }
        first = last = null;
        size = 0;
        modCount++;
    }
```

### pop 和 push

push其实就是调用addFirst(e)方法，pop调用的就是removeFirst()方法。

### toArray

创建一个Object的数组对象，然后将所有的节点都添加到Object对象中，返回Object数组对象。

```java
public Object[] toArray() {
        Object[] result = new Object[size];
        int i = 0;
        for (Node<E> x = first; x != null; x = x.next)
            result[i++] = x.item;
        return result;
    }
```

### listIterator

这个方法返回的是一个内部类ListIterator，用户可以使用这个内部类变量当前的链表元素，但是由于LinkedList也是非线程安全的类，所以和上一篇文章中的[JDK源码分析（1）ArrayList](https://blog.csdn.net/tangzongpi6936/article/details/83048126#t11)  Iterator一样，多线程下面使用，也可能会产生多线程修改的异常。

```java
public ListIterator<E> listIterator(int index) {
        checkPositionIndex(index);
        return new ListItr(index);
    }
```