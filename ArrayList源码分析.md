# ArrayList源码分析 JDK1.7

根据分析源码可知，ArrayList内部维护了一个数组elementData，根据需要对其扩容等

**添加元素-add**

ArrayList<String> list = new ArrayList<>().
在使用这个构造器创建对象的时候，将一个空数组赋给了elementData，此时size为默认值0。在添加元素的时候，会事先判断数组容量是否足够，如果内置数组为空，会将容量设置为默认值10；如果size不够，会进行扩容操作，一次扩大为原有的1.5倍，详情可见ensureCapacityInternal方法。

由此可见，一旦容量不够，就会重新创建一个更大的数组，因此在需要频繁增加元素的场景使用ArrayList作为容器，是非常浪费资源的。不过如果可以事先确定集合中会有多少元素，则可以在创建对象的时候直接指定集合大小，避免频繁创建数组。

**删除元素-remove**

首先确认传入的索引是否越界。移除的思路为，将要删除元素时候的元素全体向前移一位，使用System.arraycopy()方法。此时最后一位元素再也无法通过索引访问，及时置空，方便垃圾回收期清理。

**[ArrayList的线程不安全性](http://blog.csdn.net/u012859681/article/details/78206494)**

添加操作时，做两件事。一件是确保数组容量足够；一件是将值赋给数组指定索引的位置

1. 数组索引越界
    
    有两条线程，同时往同一个ArrayList对象里面添加元素。此时数组size为9。

    线程A在执行add方法的时候，获取size为9，执行ensureCapacityInternal方法时候需要的容量为10；与此同时，线程B也执行add方法，获取size仍为9，需要的容量也为10。AB两个线程几乎同时发现此时elementData.length的大小就是10，所以无需扩容。

    接着，执行elementData[size++] = e的赋值操作。A线程将elementData[9]的位置占了。此时size变成了10，B就要占据elementData[10]的位置，而此时索引10超出了数组索引的范围，因此会报索引越界异常。

2. 值的覆盖
    
    elementData[size++] = e。这一步操作，实际上分为两步进行：elementData[size] = e；size = size + 1。

    size为0。当A线程添加元素A的时候，将A放在了角标为0的位置；而此时B也走到这一步，size仍然为0，B也将要添加的元素B放在角标为0的位置，此举则将A值覆盖了。 而后，size会增加到2，就会出现elementData[0]为B，elementData[1]为null的情况。

验证代码如下：

```java
public static void main(String[] args) throws InterruptedException {
   final List<String> list = new ArrayList<>();
   new Thread(new Runnable() {
       @Override
       public void run() {
           for (int i = 0; i < 1000; i++) {
               list.add("A"+i);
               try {
                   Thread.sleep(1);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       }
   }).start();

   new Thread(new Runnable() {
       @Override
       public void run() {
           for (int i = 0; i < 1000; i++) {
               list.add("B"+i);
               try {
                   Thread.sleep(1);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
       }
   }).start();

    Thread.sleep(10000);

    // 打印所有结果
    for (int i = 0; i < list.size(); i++) {
        if (list.get(i)==null) {
            System.out.println("----------------------");
        }
        System.out.println("第" + (i + 1) + "个元素为：" + list.get(i));
    }
}

```
   
从1.2版本开始逐渐被替代的Vector，是线程安全，但是效率低的。查看其源码发现，它的方法基本与ArrayList一致，只是很多方法都加了锁。如下：

```java
public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized E remove(int index) {
    modCount++;
    if (index >= elementCount)
        throw new ArrayIndexOutOfBoundsException(index);
    E oldValue = elementData(index);

    int numMoved = elementCount - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--elementCount] = null; // Let gc do its work

    return oldValue;
}
```


ArrayList的源码：

```java
package java.util;

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /**
     * Default initial capacity.
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * Shared empty array instance used for empty instances.
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    /**
     * The array buffer into which the elements of the ArrayList are stored.
     * The capacity of the ArrayList is the length of this array buffer. Any
     * empty ArrayList with elementData == EMPTY_ELEMENTDATA will be expanded to
     * DEFAULT_CAPACITY when the first element is added.

     * 数据存储的数组。未指定长度
     */
    private transient Object[] elementData;

    /**
     * The size of the ArrayList (the number of elements it contains).
     *
     * @serial
     */
    private int size;

    /**
     * Constructs an empty list with the specified initial capacity.
     *
     * @param  initialCapacity  the initial capacity of the list
     * @throws IllegalArgumentException if the specified initial capacity
     *         is negative
     */
    public ArrayList(int initialCapacity) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        this.elementData = new Object[initialCapacity];
    }

    /**
     * Constructs an empty list with an initial capacity of ten.
     */
    public ArrayList() {
        super();
        this.elementData = EMPTY_ELEMENTDATA;
    }

    /**
     * Constructs a list containing the elements of the specified
     * collection, in the order they are returned by the collection's
     * iterator.
     *
     * @param c the collection whose elements are to be placed into this list
     * @throws NullPointerException if the specified collection is null
     */
    public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();
        size = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }
   
    /**
     * Appends the specified element to the end of this list.
     *
     * @param e element to be appended to this list
     * @return <tt>true</tt> (as specified by {@link Collection#add})
     */
    public boolean add(E e) {
        //
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }

    private void ensureCapacityInternal(int minCapacity) {
        //如果内置数组为空数组，将其容量定义为默认值——10
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }

    private void ensureExplicitCapacity(int minCapacity) {
        //修改次数
        modCount++;

        // overflow-conscious code
        //minCapacity是添加一个元素，所需要的最小容量
        //如果内置数组的容量小于最小容量；那么，内置数组的需要扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //位运算
        /*得到数组的旧容量，然后进行oldCapacity + (oldCapacity >> 1)，将oldCapacity 右移一位，其效果相当于oldCapacity /2，我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，*/
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果扩容后的大小仍然不够，则将最小容量赋值为最新容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //判断扩容后的容量是否超过了允许的最大值
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        //增加了容量，则将原有的元素拷贝到新创建的数组中
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    /**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     * 最大容量设置为Integer最大值减去8
     * 使用Integer最大值是因为，ArrayList中定义集合size是用的int类型。
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    private static int hugeCapacity(int minCapacity) {
        //minCapacity什么时候可能小于0？
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //将数组容量最大值赋给最新容量
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }


    /**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E remove(int index) {
        //确认索引是否超出了边界
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        //需要移动的个数
        int numMoved = size - index - 1;
        //要移除的元素不是数组中最后一个元素的情况
        if (numMoved > 0)
            //将要删除的元素后面的元素，向前移动一位
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        //将数组容量减少一个，并将数组中再也无法通过索引访问的元素置空，让垃圾回收器尽快回收
        elementData[--size] = null; // clear to let GC do its work

        return oldValue;
    }

    /**
     * Checks if the given index is in range.  If not, throws an appropriate
     * runtime exception.  This method does *not* check if the index is
     * negative: It is always used immediately prior to an array access,
     * which throws an ArrayIndexOutOfBoundsException if index is negative.
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
}

```


Arrays工具类的源码

```java
package java.util;

import java.lang.reflect.*;

public class Arrays {
    /**
     * Copies the specified array, truncating or padding with nulls (if necessary)
     * so the copy has the specified length.  For all indices that are
     * valid in both the original array and the copy, the two arrays will
     * contain identical values.  For any indices that are valid in the
     * copy but not the original, the copy will contain <tt>null</tt>.
     * Such indices will exist if and only if the specified length
     * is greater than that of the original array.
     * The resulting array is of exactly the same class as the original array.
     *
     * @param original the array to be copied
     * @param newLength the length of the copy to be returned
     * @return a copy of the original array, truncated or padded with nulls
     *     to obtain the specified length
     * @throws NegativeArraySizeException if <tt>newLength</tt> is negative
     * @throws NullPointerException if <tt>original</tt> is null
     * @since 1.6
     */
    public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }

    /**
     * Copies the specified array, truncating or padding with nulls (if necessary)
     * so the copy has the specified length.  For all indices that are
     * valid in both the original array and the copy, the two arrays will
     * contain identical values.  For any indices that are valid in the
     * copy but not the original, the copy will contain <tt>null</tt>.
     * Such indices will exist if and only if the specified length
     * is greater than that of the original array.
     * The resulting array is of the class <tt>newType</tt>.
     *
     * @param original the array to be copied
     * @param newLength the length of the copy to be returned
     * @param newType the class of the copy to be returned
     * @return a copy of the original array, truncated or padded with nulls
     *     to obtain the specified length
     * @throws NegativeArraySizeException if <tt>newLength</tt> is negative
     * @throws NullPointerException if <tt>original</tt> is null
     * @throws ArrayStoreException if an element copied from
     *     <tt>original</tt> is not of a runtime type that can be stored in
     *     an array of class <tt>newType</tt>
     * @since 1.6
     */
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        //此处的目的是创建一个扩容后的空数组，但是对于此写法的原因不是特别懂？？
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        //
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
}
```


System类的源码：

```java
package java.lang;

public final class System {
    //native关键字——java调用非java代码的接口
    public static native void arraycopy(Object src,  int  srcPos,
                                        Object dest, int destPos,
                                        int length);
}
```