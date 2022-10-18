# ArrayList

## ArrayList简介

ArrayList 是一个数组队列，相当于 动态数组。与Java中的数组相比，它的容量能动态增长。它继承于AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。
ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。
ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。
ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆。
ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输。

和Vector不同，ArrayList中的操作不是线程安全的！所以，建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

## ArrayList属性

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	//序列化ID
    private static final long serialVersionUID = 8683452581122892189L;

    //默认初始化的容量
    private static final int DEFAULT_CAPACITY = 10;

    //一个空数组实例
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //一个空数组实例，如果使用默认构造函数创建，则默认对象内容默认是该值
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    //当前数据对象存放地方，当前对象不参与序列化
    transient Object[] elementData; // non-private to simplify nested class access

    //当前数组的大小
    private int size;
    ...
```

## ArrayList构造函数

### 无参构造函数

```java
//无参构造函数
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

### 带int类型的构造函数

```java
//带int类型的构造函数
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

### 带Collection对象的构造函数

```java
//带Collection对象的构造函数
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            if (elementData.getClass() != Object[].class)
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```

## add方法

### add(E e)

add主要的执行逻辑如下：
1）确保数组已使用长度（size）加1之后足够存下 下一个数据
2）修改次数modCount 标识自增1，如果当前数组已使用长度（size）加1后的大于当前的数组长度，则调用grow方法，增长数组，grow方法会将当前数组的长度变为原来容量的1.5倍。
3）确保新增的数据有地方存储之后，则将新元素添加到位于size的位置上。
4）返回添加成功布尔值。

```java
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
将修改次数（modCount）自增1，判断是否需要扩充数组长度,判断条件就是用当前所需的数组最小长度与数组的长度对比，如果大于0，则增长数组长度。

ensureCapacityInternal

```java
private void ensureCapacityInternal(int minCapacity) {
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
```
calculateCapacity
```java
private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```
将修改次数（modCount）自增1，判断是否需要扩充数组长度,判断条件就是用当前所需的数组最小长度与数组的长度对比，如果大于0，则增长数组长度。

ensureExplicitCapacity

```java
private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```
如果当前的数组已使用空间（size）加1之后 大于数组长度，则增大数组容量，扩大为原来的1.5倍。

grow

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

### add(int index, E e)

这个方法其实和上面的add类似，该方法可以按照元素的位置，指定位置插入元素，具体的执行逻辑如下：
1）确保数插入的位置小于等于当前数组长度，并且不小于0，否则抛出异常
2）确保数组已使用长度（size）加1之后足够存下 下一个数据
3）修改次数（modCount）标识自增1，如果当前数组已使用长度（size）加1后的大于当前的数组长度，则调用grow方法，增长数组
4）grow方法会将当前数组的长度变为原来容量的1.5倍。
5）确保有足够的容量之后，使用System.arraycopy 将需要插入的位置（index）后面的元素统统往后移动一位。
6）将新的数据内容存放到数组的指定位置（index）上

```java
public void add(int index, E e) {
            rangeCheckForAdd(index);
            checkForComodification();
            parent.add(parentOffset + index, e);
            this.modCount = parent.modCount;
            this.size++;
        }
```

## get方法

返回指定位置上的元素

```java
public E get(int index) {
            rangeCheck(index);
            checkForComodification();
            return ArrayList.this.elementData(offset + index);
        }
```

## set方法

确保set的位置小于当前数组的长度（size）并且大于0，获取指定位置（index）元素，然后放到oldValue存放，将需要设置的元素放到指定的位置（index）上，然后将原来位置上的元素oldValue返回给用户。

```java
public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

## contains方法

调用indexOf方法，遍历数组中的每一个元素作对比，如果找到对于的元素，则返回true，没有找到则返回false。

```java
public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```

indexOf

```java
public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```

## remove方法

### 根据索引remove

1）判断索引有没有越界

2）自增修改次数

3）将指定位置（index）上的元素保存到oldValue

4）将指定位置（index）上的元素都往前移动一位

5）将最后面的一个元素置空，好让垃圾回收器回收

6）将原来的值oldValue返回

```java
public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }
```

### 根据对象

```java
public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```

定位到需要remove的元素索引，先将index后面的元素往前面移动一位（调用System.arraycooy实现），然后将最后一个元素置空。

fastRemove

```java
private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```

## clear方法

添加操作次数（modCount），将数组内的元素都置空，等待垃圾收集器收集，不减小数组容量。

```java
public void clear() {
        modCount++;

        // clear to let GC do its work
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

## subList方法

我们看到代码中是创建了一个ArrayList 类里面的一个内部类SubList对象，传入的值中第一个参数时this参数，其实可以理解为返回当前list的部分视图，真实指向的存放数据内容的地方还是同一个地方，如果修改了sublist返回的内容的话，那么原来的list也会变动。

```java
public List<E> subList(int fromIndex, int toIndex) {
        subListRangeCheck(fromIndex, toIndex, size);
        return new SubList(this, 0, fromIndex, toIndex);
    }
```

## trimTosize方法

1）修改次数加1
2）将elementData中空余的空间（包括null值）去除，例如：数组长度为10，其中只有前三个元素有值，其他为空，那么调用该方法之后，数组的长度变为3.

```java
public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```

## iterator方法

interator方法返回的是一个内部类，由于内部类的创建默认含有外部的this指针，所以这个内部类可以调用到外部类的属性。

```java
public Iterator<E> iterator() {
        return new Itr();
    }
```

这里转发别人的：

一般的话，调用完iterator之后，我们会使用iterator做遍历，这里使用next做遍历的时候有个需要注意的地方，就是调用next的时候，可能会引发ConcurrentModificationException，当修改次数，与期望的修改次数（调用iterator方法时候的修改次数）不一致的时候，会发生该异常，详细我们看一下代码实现：

```java
@SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
```

expectedModCount这个值是在用户调用ArrayList的iterator方法时候确定的，但是在这之后用户add，或者remove了ArrayList的元素，那么modCount就会改变，那么这个值就会不相等，将会引发ConcurrentModificationException异常，这个是在多线程使用情况下，比较常见的一个异常。
```java
final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```
## 序列化
ArrayList基于数组实现，并且具有动态扩容特性，因此保存元素的数组不一定都会被使用，那么就没必要全部进行序列化。

保存元素的数组elementData使用transient修饰，该关键字声明数组默认不会被序列化
```java
transient Object[] elementData; // non-private to simplify nested class access
```

ArrayList实现了writeObject()和readObject()来控制只序列化数组中有元素填充那部分内容。
```java
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

```java
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            int capacity = calculateCapacity(elementData, size);
            SharedSecrets.getJavaOISAccess().checkArray(s, Object[].class, capacity);
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```
序列化时需要使用ObjectOuTputStream的writeObject()将对象转换为字节流并输出。而writeObject()方法在传入的对象存在writeObject()的时候会去反射调用该对象的writeObject()来实现序列化。反序列化使用的是ObjectInputStream的readObject()方法，原理类似。
```java
ArrayList list = new ArrayList(); ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file)); oos.writeObject(list);
```

## 小结

ArrayList总体来说比较简单，不过ArrayList还有以下一些特点：

- ArrayList自己实现了序列化和反序列化的方法，因为它自己实现了 private void writeObject(java.io.ObjectOutputStream s)和 private void readObject(java.io.ObjectInputStream s) 方法
- ArrayList基于数组方式实现，无容量的限制（会扩容）
- 添加元素时可能要扩容（所以最好预判一下），删除元素时不会减少容量（若希望减少容量，trimToSize()），**删除元素时，将删除掉的位置元素置为null，下次gc就会回收这些元素所占的内存空间**。
- 线程不安全
- add(int index, E element)：添加元素到数组中指定位置的时候，需要将该位置及其后边所有的元素都整块向后复制一位
- get(int index)：获取指定位置上的元素时，可以通过索引直接获取（O(1)）
- remove(Object o)需要遍历数组
- remove(int index)不需要遍历数组，只需判断index是否符合条件即可，效率比remove(Object o)高
- contains(E)需要遍历数组
- 使用iterator遍历可能会引发多线程异常