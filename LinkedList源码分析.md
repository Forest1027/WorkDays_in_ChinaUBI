# LinkedList源码分析 JDK1.7
LinkedList是双向链表结构，内部维护一个Node私有静态对象。JDK1.7的链表是线状链表，有头和尾，并非环状。

**添加元素**

LinkedList在添加元素的时候会先判断要插入的位置是否是头或尾。如果是头或尾，直接获取first或last的原本的头部尾部元素，修改Node的prev和next变量进行拼接。

如果不是头或尾，则使用二分法遍历链表，找到指定索引的元素A，并且获取此元素的prev节点B。将B的next节点设置为要插入的target节点；将target节点的prev节点设置为B，next节点设置为A；将A的prev节点设置为target。如果要插入的是一个集合，则遍历待插入集合进行操作。

**删除元素**

LinkedList采用的是队列结构，因此是先进先出。添加是往尾部添加，删除是将头部删除，也可以将尾部删除或指定索引元素删除。

**LinkedList与ArrayList相比较**

LinkedList增删的时候，是将节点关联的前后元素引用修改，在修改头尾元素以外的情况还需要用二分法遍历链表；而ArrayList增删时都调用了System.arraycopy方法，扩容时还需要重新创建数组。

LinkedList查找元素时，是遍历节点，一一比对；ArrayList查找元素时是直接通过索引访问。

如下数据，当循环次数达到一百万的时候，LinkedList的添加方法所用时间才较为稳定的小于ArrayList（十万到百万之间的测试次数较少，一百万是否为临界点尚不清楚，也不知道是不是电脑的问题）。

查询方法两者差异极大，ArrayList显著优于LinkedList

*1000次：*

    arraylist(add)用时：1
    linkedlist(add)用时：1
    arraylist(get)用时：0
    linkedlist(get)用时：8
    arraylist(remove)用时：0
    linkedlist(remove)用时：2

*10000次：*

    arraylist(add)用时：4
    linkedlist(add)用时：5
    arraylist(get)用时：1
    linkedlist(get)用时：251
    arraylist(remove)用时：2
    linkedlist(remove)用时：2

*100000次：*

    arraylist(add)用时：11
    linkedlist(add)用时：12
    arraylist(get)用时：5
    linkedlist(get)用时：7884
    arraylist(remove)用时：4
    linkedlist(remove)用时：10

*1000000次：*

    arraylist(add)用时：245
    linkedlist(add)用时：104
    arraylist(get)用时：8
    linkedlist(get)用时：1635594
    arraylist(remove)用时：7
    linkedlist(remove)用时：19



测试代码如下：

```java
public static void main(String[] args) {
    ArrayList<Integer> arrayList = new ArrayList<>();
    LinkedList<Integer> linkedList = new LinkedList<>();
    int times = 1000;

    //添加
    long start = System.currentTimeMillis();
    for (int i = 0; i < times; i++) {
        arrayList.add(i);
    }
    long end = System.currentTimeMillis();
    System.out.println("arraylist(add)用时：" + (end - start));

    start = System.currentTimeMillis();
    for (int j = 0; j < times; j++) {
        linkedList.add(j);
    }
    end = System.currentTimeMillis();
    System.out.println("linkedlist(add)用时：" + (end - start));

    //查询
    start = System.currentTimeMillis();
    for (int i = 0; i < times; i++) {
        arrayList.get(i);
    }
    end = System.currentTimeMillis();
    System.out.println("arraylist(get)用时：" + (end - start));

    start = System.currentTimeMillis();
    for (int i = 0; i < times; i++) {
        linkedList.get(i);
    }
    end = System.currentTimeMillis();
    System.out.println("linkedlist(get)用时：" + (end - start));

    //移除
    start = System.currentTimeMillis();
    for (int i = times - 1; i >= 0; i--) {
        arrayList.remove(i);
    }
    end = System.currentTimeMillis();
    System.out.println("arraylist(remove)用时：" + (end - start));

    start = System.currentTimeMillis();
    for (int i = times - 1; i >= 0; i--) {
        linkedList.remove(i);
    }
    end = System.currentTimeMillis();
    System.out.println("linkedlist(remove)用时：" + (end - start));

}
```


LinkedList源码:

```java
/**
 * Appends the specified element to the end of this list.
 *
 * <p>This method is equivalent to {@link #addLast}.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/**
 * Links e as last element.
 */
void linkLast(E e) {
    //将新添加的元素加入链表的末端
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    //维护内部定义的指针变量
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}

/**
 * LinkedList内置了一个Node私有静态类
 * 每一个节点都包含对前一个以及后一个节点的引用
 */
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}

/**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
 transient Node<E> last;

/**
 * Inserts all of the elements in the specified collection into this
 * list, starting at the specified position.  Shifts the element
 * currently at that position (if any) and any subsequent elements to
 * the right (increases their indices).  The new elements will appear
 * in the list in the order that they are returned by the
 * specified collection's iterator.
 *
 * @param index index at which to insert the first element
 *              from the specified collection
 * @param c collection containing elements to be added to this list
 * @return {@code true} if this list changed as a result of the call
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    //确认索引是否越界
    checkPositionIndex(index);

    //判断要插入的集合是否为空
    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;

    Node<E> pred, succ;
    if (index == size) {
        //构造方法调用时，index与size相等
        //此时链表为空
        succ = null;
        pred = last;
    } else {
        //链表非空时，返回index对应的元素
        succ = node(index);
        pred = succ.prev;
    }

    //插入传入的集合元素
    for (Object o : a) {
        //此注解用于禁止非受检警告
        @SuppressWarnings("unchecked") E e = (E) o;
        //取到每一个元素，创建为新的节点
        Node<E> newNode = new Node<>(pred, e, null);
        //与已有节点建立联系
        if (pred == null)
            //如果链表为空，那么第一个元素就是新创建的节点
            //只有在传入集合参数创建对象时，会走到这个判断里面，并且也只会在第一次创建节点时走一次
            first = newNode;
        else
            //如果链表不为空
            pred.next = newNode;
        pred = newNode;
    }

    if (succ == null) {
        //如果succ为空，说明链表原本没有元素。那么last就为最后插入的那个节点
        last = pred;
    } else {
        //如果succ不为空，将pred和succ之间建立联系
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}

/**
 * Returns the (non-null) Node at the specified element index.
 * 找到index位上的元素
 */
Node<E> node(int index) {
    // assert isElementIndex(index);
    // 二分法查找，提高查询效率
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

/**
 * Retrieves and removes the head (first element) of this list.
 *
 * @return the head of this list
 * @throws NoSuchElementException if this list is empty
 * @since 1.5
 */
public E remove() {
    //链表移除元素是移除掉第一个元素
    return removeFirst();
}

/**
 * Removes and returns the first element from this list.
 *
 * @return the first element from this list
 * @throws NoSuchElementException if this list is empty
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}

/**
 * Unlinks non-null first node f.
 */
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