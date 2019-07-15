# LinkedHashMap
## 概述
`LinkedHashMap`是一个关联数组、哈希表，它是**线程不安全**的，允许**key为null,value为null**。他继承自`HashMap`，实现了`Map<K,V>`接口。其内部还维护了一个**双向链表**，在每次**插入数据，或者访问、修改数据**时，**会增加节点、或调整链表的节点顺序**。以决定迭代时输出的顺序。
默认情况，遍历时的顺序是按照插入节点的顺序。这也是其与`HashMap`最大的区别。也可以在构造时传入`accessOrder`参数，使得其遍历顺序**按照访问的顺序**输出。
## 构造函数
```java
    //默认是false，则迭代时输出的顺序是插入节点的顺序。若为true，则输出的顺序是按照访问节点的顺序。
    //为true时，可以在这基础之上构建一个LruCach
    final boolean accessOrder;

    public LinkedHashMap() {
        super();
        accessOrder = false;
    }
    //指定初始化时的容量，
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }
    //指定初始化时的容量，和扩容的加载因子
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }
    //指定初始化时的容量，和扩容的加载因子，以及迭代输出节点的顺序
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
    //利用另一个Map 来构建，
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        //该方法上文分析过，批量插入一个map中的所有数据到 本集合中。
        putMapEntries(m, false);
    }

```
**小结**：
构造函数和`HashMap`相比，就是增加了一个`accessOrder`参数。用于控制迭代时的节点顺序。
## 节点
`LinkedHashMap`的节点`Entry<K,V>`继承自`HashMap<K,V>`，在其基础上扩展了一下。改成了一个**双向链表**。
```java
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```
同时类里有两个成员变量`head tail`，分别指向内部双向链表的表头、表尾。
```java
    //双向链表的头结点
    transient LinkedHashMap.Entry<K,V> head;

    //双向链表的尾节点
    transient LinkedHashMap.Entry<K,V> tail;
```
## 增
`LinkedHashMap`并没有重写任何put方法。但是其重写了构建新节点的`newNode()`方法. 
`newNode()`会在`HashMap`的`putVal()`方法里被调用，`putVal()`方法会在批量插入数据`putMapEntries(Map<? extends K, ? extends V> m, boolean evict)`或者插入单个数据`public V put(K key, V value)`时被调用。

`LinkedHashMap`重写了`newNode()`,在每次**构建新节点**时，通过`linkNodeLast(p)`;将**新节点链接在内部双向链表的尾部**。
```java
    //在构建新节点时，构建的是`LinkedHashMap.Entry` 不再是`Node`.
    Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }
    //将新增的节点，连接在链表的尾部
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        //集合之前是空的
        if (last == null)
            head = p;
        else {//将新节点连接在链表的尾部
            p.before = last;
            last.after = p;
        }
    }
```
以及`HashMap`专门预留给`LinkedHashMap`的`afterNodeAccess() afterNodeInsertion() afterNodeRemoval()`方法。
```java
    // Callbacks to allow LinkedHashMap post-actions
    void afterNodeAccess(Node<K,V> p) { }
    void afterNodeInsertion(boolean evict) { }
    void afterNodeRemoval(Node<K,V> p) { }
        //回调函数，新节点插入之后回调 ， 根据evict 和   判断是否需要删除最老插入的节点。如果实现LruCache会用到这个方法。
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        //LinkedHashMap 默认返回false 则不删除节点
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
    //LinkedHashMap 默认返回false 则不删除节点。 返回true 代表要删除最早的节点。通常构建一个LruCache会在达到Cache的上限是返回true
    protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
        return false;
    }
```
`void afterNodeInsertion(boolean evict)`以及`boolean removeEldestEntry(Map.Entry<K,V> eldest)`是构建LruCache需要的回调，在`LinkedHashMap`里可以忽略它们。
## 删
`LinkedHashMap`也没有重写`remove()`方法，因为它的删除逻辑和`HashMap`并无区别。 
但它重写了`afterNodeRemoval()`这个回调方法。该方法会在`Node<K,V> removeNode(int hash, Object key, Object value, boolean matchValue, boolean movable)`方法中回调，`removeNode()`会在所有涉及到删除节点的方法中被调用，是删除节点操作的真正执行者。
```java
    //在删除节点e时，同步将e从双向链表上删除
    void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        //待删除节点 p 的前置后置节点都置空
        p.before = p.after = null;
        //如果前置节点是null，则现在的头结点应该是后置节点a
        if (b == null)
            head = a;
        else//否则将前置节点b的后置节点指向a
            b.after = a;
        //同理如果后置节点时null ，则尾节点应是b
        if (a == null)
            tail = b;
        else//否则更新后置节点a的前置节点为b
            a.before = b;
    }
```
## 查
`LinkedHashMap`重写了`get()`和`getOrDefault()`方法：
```java
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
    public V getOrDefault(Object key, V defaultValue) {
       Node<K,V> e;
       if ((e = getNode(hash(key), key)) == null)
           return defaultValue;
       if (accessOrder)
           afterNodeAccess(e);
       return e.value;
   }
```
对比`HashMap`中的实现,`LinkedHashMap`只是增加了在成员变量(构造函数时赋值)`accessOrder`为true的情况下，要去回调`void afterNodeAccess(Node<K,V> e)`函数。
```java
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```
在`afterNodeAccess()`函数中，**会将当前被访问到的节点e，移动至内部的双向链表的尾部**。
```java
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;//原尾节点
        //如果accessOrder 是true ，且原尾节点不等于e
        if (accessOrder && (last = tail) != e) {
            //节点e强转成双向链表节点p
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            //p现在是尾节点， 后置节点一定是null
            p.after = null;
            //如果p的前置节点是null，则p以前是头结点，所以更新现在的头结点是p的后置节点a
            if (b == null)
                head = a;
            else//否则更新p的前直接点b的后置节点为 a
                b.after = a;
            //如果p的后置节点不是null，则更新后置节点a的前置节点为b
            if (a != null)
                a.before = b;
            else//如果原本p的后置节点是null，则p就是尾节点。 此时 更新last的引用为 p的前置节点b
                last = b;
            if (last == null) //原本尾节点是null  则，链表中就一个节点
                head = p;
            else {//否则 更新 当前节点p的前置节点为 原尾节点last， last的后置节点是p
                p.before = last;
                last.after = p;
            }
            //尾节点的引用赋值成p
            tail = p;
            //修改modCount。
            ++modCount;
        }
    }
```
值得注意的是，`afterNodeAccess()`函数中，会修改`modCount`,因此当你正在`accessOrder=true`的模式下,迭代`LinkedHashMap`时，如果同时查询访问数据，也会导致`fail-fast`，因为迭代的顺序已经改变。
## containsValue
它重写了该方法，相比`HashMap`的实现，**更为高效**。
```java
    public boolean containsValue(Object value) {
        //遍历一遍链表，去比较有没有value相等的节点，并返回
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
            V v = e.value;
            if (v == value || (value != null && value.equals(v)))
                return true;
        }
        return false;
    }
```
对比`HashMap`，是用两个for循环遍历，相对低效。
```java
    public boolean containsValue(Object value) {
        Node<K,V>[] tab; V v;
        if ((tab = table) != null && size > 0) {
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                    if ((v = e.value) == value ||
                        (value != null && value.equals(v)))
                        return true;
                }
            }
        }
        return false;
    }
```
## 遍历
重写了`entrySet()`如下：
```java
    public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es;
        //返回LinkedEntrySet
        return (es = entrySet) == null ? (entrySet = new LinkedEntrySet()) : es;
    }
    final class LinkedEntrySet extends AbstractSet<Map.Entry<K,V>> {
        public final Iterator<Map.Entry<K,V>> iterator() {
            return new LinkedEntryIterator();
        }
    }
```
最终的EntryIterator:
```java
    final class LinkedEntryIterator extends LinkedHashIterator
        implements Iterator<Map.Entry<K,V>> {
        public final Map.Entry<K,V> next() { return nextNode(); }
    }

    abstract class LinkedHashIterator {
        //下一个节点
        LinkedHashMap.Entry<K,V> next;
        //当前节点
        LinkedHashMap.Entry<K,V> current;
        int expectedModCount;

        LinkedHashIterator() {
            //初始化时，next 为 LinkedHashMap内部维护的双向链表的扁头
            next = head;
            //记录当前modCount，以满足fail-fast
            expectedModCount = modCount;
            //当前节点为null
            current = null;
        }
        //判断是否还有next
        public final boolean hasNext() {
            //就是判断next是否为null，默认next是head  表头
            return next != null;
        }
        //nextNode() 就是迭代器里的next()方法 。
        //该方法的实现可以看出，迭代LinkedHashMap，就是从内部维护的双链表的表头开始循环输出。
        final LinkedHashMap.Entry<K,V> nextNode() {
            //记录要返回的e。
            LinkedHashMap.Entry<K,V> e = next;
            //判断fail-fast
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            //如果要返回的节点是null，异常
            if (e == null)
                throw new NoSuchElementException();
            //更新当前节点为e
            current = e;
            //更新下一个节点是e的后置节点
            next = e.after;
            //返回e
            return e;
        }
        //删除方法 最终还是调用了HashMap的removeNode方法
        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
    }
```
值得注意的就是：`nextNode()`就是迭代器里的`next()`方法 。 
该方法的实现可以看出，迭代`LinkedHashMap`，就是从**内部维护的双链表的表头开始循环输出**。 
而双链表节点的顺序在LinkedHashMap的**增、删、改、查时都会更新。以满足按照插入顺序输出，还是访问顺序输出**。
## 总结
`LinkedHashMap`相对于`HashMap`的源码比，是很简单的。因为大树底下好乘凉。它继承了`HashMap`，仅重写了几个方法，以**改变它迭代遍历时的顺序**。这也是其与`HashMap`相比最大的不同。 
在每次**插入数据，或者访问、修改数据时，会增加节点、或调整链表的节点顺序**。以决定迭代时输出的顺序。
- `accessOrder`,默认是false，则迭代时输出的顺序是**插入节点的顺序**。若为true，则输出的顺序是按照访问节点的顺序。为true时，可以在这基础之上构建一个`LruCache`.
- `LinkedHashMap`并没有重写任何put方法。但是其重写了构建新节点的newNode()方法.在每次构建新节点时，将**新节点链接在内部双向链表的尾部**.
- `accessOrder=true`的模式下,在`afterNodeAccess()`函数中，会将当前**被访问**到的节点e，**移动**至内部的双向链表**的尾部**。值得注意的是，`afterNodeAccess()`函数中，会修改`modCount`,因此当你正在`accessOrder=true`的模式下,迭代`LinkedHashMap`时，如果同时查询访问数据，也会导致`fail-fast`，因为迭代的顺序已经改变.
- `nextNode()`就是迭代器里的`next()`方法。该方法的实现可以看出，迭代`LinkedHashMap`，就是从**内部维护的双链表的表头开始循环输出**。而双链表节点的顺序在`LinkedHashMap`的**增、删、改、查时都会更新。以满足按照插入顺序输出，还是访问顺序输出**.
- 它与`HashMap`比，还有一个小小的优化，重写了`containsValue()`方法，直接遍历内部链表去比对value值是否相等.
