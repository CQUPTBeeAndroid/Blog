# Java集合源码分析（一）

**author**：王川云

标签（空格分隔）： 未分类

---
![image.png](https://upload-images.jianshu.io/upload_images/13507391-9f99b113124e5c16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





## 一、集合的继承关系

Collection是继承自Iterable

## 二、什么是Iterable

1 . Iterable是可迭代的意思
2 . Iterable是一个接口
3 . 我们看一下官方解释：Implementing this interface allows an object to be the target of the "for-each loop" statement.
简单意思就是：实现此接口，让对象允许成为"for-each loop"的目标。
相信什么是"for-each"大家都知道

4 . 里面有三个方法：

* 1 Iterator<T> iterator()；生成一个迭代器

```
Iterator<T> iterator();
```
* 2 default void forEach(Consumer<? super T> action)；根据action对内部元素进行遍历

```
 default void forEach(Consumer<? super T> action) {
        //判断动作是否为空
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

* 3 创建一个分割器（Spliterator），这里先不了解。
```
default Spliterator<T> spliterator() {
    return Spliterators.spliteratorUnknownSize(iterator(), 0);
}
```

那么我们简单定义一下，实现此接口的类会成为一个可迭代的容器。

## 三、什么是Collection

1 . 通过注释我们可以知道Collection是所有集合的根接口
2 . Collection中包含了很多处理容器中各种元素的方法，比如：增加，移除，比较（两个集合），转换（数组或者其他）

那么我们可以简单定义一下，这个Collection接口就是给实现它的容器提供基本规范的方法；

## 四、什么是List

1 .List是List家族的顶级接口
2 .List是一个有序集合序列（在我看来线性表）
3 .从List中的各种方法看来，List就是用来规范List家族的接口
4 .List家族的成员都必须实现这个接口，实现其中的方法，成为List的一员
5 .List家族有四个成员ArrayList、LinkList、Victor、Stack

这里简单总结一下，这些接口都是用于规范的，并没有实现具体逻辑。

## 五、ArryList是如何实现的

接下来我们要看ArryList的源码了，但是在看源码之前我们要理清思路，我们重点看什么？


*  1 ArryList的重要方法，添加，移除，扩张，比较等等还有与其相关的属性
*  2 ArryList的迭代器


1 . ArryList的迭代器，ArryList中有一个内部类专门实现了迭代器接口

```
 private class Itr implements Iterator<E> {}
```

2 . 虽然List已经有了一个基本的迭代器，但是List对这个基本的迭代器的功能并不满足，所以下面有产生了一个List专门用的迭代器：

```
private class ListItr extends Itr implements ListIterator<E> {}
```

3、初此之外，ArrayList覆写了父类和List接口的方法，添加，移除，扩张，比较等等。

* 1 ArrayList是通过数组实现
* 2 ArrayList扩张是数组的copyOf()方法，第一次扩张数组大小变为10，后面如果要扩张的话，大小变为以前的1.5倍
* 3 对ArrayList的中操作迭代器会改变ArrayList的成员变量，但是如果只对ArrayList操作，则没有影响迭代器
* 4 ArrayList里面的元素是无序的（没有按大小顺序排列），他是根据插入元素的先后顺序排列的

### 源码分析

ArrayList的源码比较清晰，这里只讨论是如何自动扩张的。
我们知道ArrayList是基于数组的，数组有什么不好的地方呢？对了，就是一旦生成了就大小固定无法扩展了，那么ArrayList是如何自动扩张的呢？
首先看向add()方法：

```
 public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
这里有一个调用了ensureCapacityInternal()方法，翻译一下：确保容量内部，那这个方法是干什么的呢？我们跟踪查看一下。

```
private void ensureCapacityInternal(int minCapacity) {
        //判断当前使用的数组是否为空数组
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        //最小的容量为DEFAULT_CAPACITY和size+1的更大者
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
```

从第二行看起，elementData是一个全局变量，用来表示ArrayList当前正在使用的数组，当前正在使用的数组是否是空数组，如果是空数组的话，最小容量就是DEFAULT_CAPACITY（10）和size+1的更大者。
好了看看第六行，又调用了一个ensureExplicitCapacity（）方法，解释一下：确保明确的容量，我们再跟踪查看，等会儿再回来。

```
 private void ensureExplicitCapacity(int minCapacity) {
 //ArrayList的结构变化增加了一次
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
        //一定会执行
            grow(minCapacity);
    }
```

modCount是什么？ ArrayList 发生结构变化的次数。
接下来看看第6行：如果最小容量大于当前正在使用的数组的容量，就执行 grow（）方法，我们继续追踪grow()方法

```
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //新的容量是旧的容量的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
        // 新容量小于最小容量的话，就把最小容量赋值给新容量
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
        //新容量大于数组最大容量，就把整数的最大值赋值给新容量
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
重点来了，看向第13行，数组实际扩张的办法，通过Arrays的静态方法copyOf（）。我们追踪一下这个方法：
```
 public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }
```
啊哈，又调用了它的三个参数的重载方法，我们继续追踪：

```
  public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }

```
完成了，就是直接new一个新的数组，然后再把旧的数组的数据复制回去。

接下来我们要返回到add（）方法中完成最后的步骤了：

```
 public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
给当前正在使用的数elementData添加元素，这里有个问题，size中的是什么呢？大家可以猜猜哦。。


## 六、LinkList是如何实现的

我们先看一下LinkList的继承关系
```
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable{}
```
我们可以看见LinkList比ArrayList多实现了一个接口Deque<E>，那么这个接口是什么呢？我们来看看
```
public interface Deque<E> extends Queue<E> {}
```
通过注释我们可以发现这个接口是一个线性双端队列，它支持容量限制的双端队列。
在这里我们对LinkList有了一点了解。接下来我们看看LinkList的注释。
```
 * Doubly-linked list implementation of the {@code List} and {@code Deque}
 * interfaces.  Implements all optional list operations, and permits all
 * elements (including {@code null}).
```
大致意思是：LinkList是一个双链表，可以操作所有元素

1、LinkList与ArrayList不同，他是双链表
2、LinkList的扩张与删除就是结点的增加和删除
3、集合中的元素是也是无序的，他是根据插入结点的位置来排列的
4、判断是从头结点插入还是从尾结点开始插入做了优化

### 源码分析

为什么说LinkList是双链表呢？

```
 /**
     * Pointer to first node.
     * Invariant: (first == null && last == null) ||
     *            (first.prev == null && first.item != null)
     */
    transient Node<E> first;

    /**
     * Pointer to last node.
     * Invariant: (first == null && last == null) ||
     *            (last.next == null && last.item != null)
     */
    transient Node<E> last;
```

LinkList定义了一个头结点和一个尾结点。Node是LinkList的内部类，我们看看他这个结点是怎么实现的：

```
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
```

首先Node是一个私有的静态泛型类，它有三个全局变量，第一个变量是结点储存的元素，后面两个变量分别是当前结点的前驱结点和后继结点，很简单。

接下来我们看看LinkList的插入结点的方法add（）

```
public boolean add(E e) {
//直接插入默认插入到尾部
        linkLast(e);
        return true;
    }
    
      public void add(int index, E element) {
      //检查这个索引是否超出链表的边界
        checkPositionIndex(index);

        if (index == size)
        //链表为空的时候或者想要插入到尾部的时候直接插入到尾部
            linkLast(element);
        else
            linkBefore(element, node(index));
    }
```

我们可以看见 LinkList的add（）有两个重载方法。

直接插入话，默认是插入到尾部的。

如果过通过索引插入的话，我们看向第15行：
这里执行了linkBefore（）方法，插入之前的查找工作，传入了需要插入的元素和node（）这个方法的返回值
我们先看看node（）这个方法是干什么的：

```
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

这里代码很简单，如果请求插入的索引小于size的一半则从头开始遍历，反之则从尾部开始遍历。返回的也是一个节点

然后我们看看linkBefore（）方法：

```
 /**
     * Inserts element e before non-null Node succ.//把我们要插入的元素插入到suuc这个节点之前
     */
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
这部分代码的意思就是把我们需要插入的节点插入到我们用node（）方法找到的节点的前一个节点那里。

好了LinkList的源码主要也是分析add（）方法。

## Set

