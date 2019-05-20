#### 1.1、ArrayList
ArrayList的数据结构就是数组，数组元素类型为Object类型，即可以存放所有类型数据

ArrayList的父类：
- ArrayList extends AbstractList
- AbstractList extends AbstractCollection 

ArrayList实现了哪些接口？

- List<E>接口：但是ArrayList的父类AbstractList也实现了List<E>接口

- RandomAccess接口：这个是一个标记性接口，通过查看api文档，它的作用就是用来快速随机存取，有关效率的问题，在实现了该接口的话，那么使用普通的for循环来遍历，性能更高，例如arrayList。

- Cloneable接口：实现了该接口，就可以使用Object.Clone()方法了。

- Serializable接口：实现该序列化接口，表明该类可以被序列化，什么是序列化？简单的说，就是能够从类变成字节流传输，然后还能从字节流变成原来的类。

##### 先看看ArrayList的一些静态常量吧：


```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 版本号
    private static final long serialVersionUID = 8683452581122892189L;
    // 缺省容量
    private static final int DEFAULT_CAPACITY = 10;
    // 空对象数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    // 缺省空对象数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    // 默认数组
    transient Object[] elementData;
    // 实际元素大小，默认为0
    private int size;
    // 最大数组容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
}

```

##### 构造方法
- 无参构造
```
　//ArrayList中储存数据的其实就是一个数组，这个数组就是前面的elementData
　　 public ArrayList() {　　
        super();        //调用父类中的无参构造方法，父类中的是个空的构造方法
        this.elementData = EMPTY_ELEMENTDATA;//将elementData初始化，elementData也是个Object[]类型。空的Object[]等会的默认大小为10
    }
```

-有参构造1

```
 public ArrayList(int initialCapacity) {
        super(); //父类中空的构造方法
        if (initialCapacity < 0)    //判断如果自定义大小的容量小于0，就抛出下面这个据异常
            throw new IllegalArgumentException("Illegal Capacity: "+   initialCapacity);
        this.elementData = new Object[initialCapacity]; //将自定义的容量大小当成初始化elementData的大小
    }
```
- 有参构造2

```
 public ArrayList(Collection<? extends E> c) {
        elementData = c.toArray();    //转换为数组
        size = elementData.length;   //数组中的数据个数
        if (elementData.getClass() != Object[].class) //每个集合的toarray()的实现方法不一样，所以需要判断一下，如果不是Object[].class类型，那么就需要使用ArrayList中的copyOf方法去改变一下。
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    }　　　
```

##### add()方法
```
 public boolean add(E e) {    
    //确定内部容量是否够了，size是数组中数据的个数，因为要添加一个元素，所以size+1，先判断size+1的这个个数数组能否放得下，就在这个方法中去判断是否数组.length是否够用了。
        ensureCapacityInternal(size + 1);  // Increments modCount!!
     //在数据中正确的位置上放上元素e，并且size++
        elementData[size++] = e;
        return true;
    }
```
我们进入ensureCapacityInternal（）方法，看看是怎么确定内部容量的：

```
private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) { //看，判断初始化的elementData是不是空的数组，也就是没有长度
    //因为如果是空的话，再进一步判断，把minCapacity赋值为DEFAULT_CAPACITY（10）和minCapacity的最大值，也就是10，还没有真正的初始化这个elementData的大小。
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
    //确认实际的容量，上面只是将minCapacity=10，这个方法就是真正的判断elementData是否够用
        ensureExplicitCapacity(minCapacity);
    }
```
进入ensureExplicitCapacity（）方法：

```
private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
//minCapacity如果大于了实际elementData的长度，那么就说明elementData数组的长度不够用，不够用那么就要增加elementData的length。那么，这里的minCapacity到底是多少呢？

/*第一种情况：由于elementData初始化时是空的数组，那么第一次add的时候，minCapacity=size+1；也就minCapacity=1，在上一个方法(确定内部容量ensureCapacityInternal)就会判断出是空的数组，就会给
　　将minCapacity=10，到这一步为止，还没有改变elementData的大小。
　第二种情况：elementData不是空的数组了，那么在add的时候，minCapacity=size+1；也就是minCapacity代表着elementData中增加之后的实际数据个数，拿着它判断elementData的length是否够用，如果length
不够用，那么肯定要扩大容量，不然增加的这个元素就会溢出。
*/
        if (minCapacity - elementData.length > 0)
    //arrayList能自动扩展大小的关键方法就在这里了
            grow(minCapacity);
    }
```
进入grow（）方法查看如何扩展数组的：

```
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;  //将扩充前的elementData大小给oldCapacity
        int newCapacity = oldCapacity + (oldCapacity >> 1);//newCapacity就是1.5倍的oldCapacity
        if (newCapacity - minCapacity < 0)//这句话就是适应于elementData就空数组的时候，length=0，那么oldCapacity=0，newCapacity=0，所以这个判断成立，在这里就是真正的初始化elementData的大小了，就是为10.前面的工作都是准备工作。
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)//如果newCapacity超过了最大的容量限制，就调用hugeCapacity，也就是将能给的最大值给newCapacity
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
    //新的容量大小已经确定好了，就copy数组，改变容量大小了。
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```
##### void add(int，E)；在特定位置添加元素，也就是插入元素


```
public void add(int index, E element) {
        rangeCheckForAdd(index);//检查index也就是插入的位置是否合理。

//跟上面的分析一样，用来确定内部容量
        ensureCapacityInternal(size + 1);  // Increments modCount!!
//这个方法就是用来在插入元素之后，要将index之后的元素都往后移一位，
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
//在目标位置上插入元素
        elementData[index] = element;
        size++;//size增加1
    } 
```
##### remove(int)方法


```
  public E remove(int index) {
  ////检查index的合理性
            rangeCheck(index);
           //删除该元素 checkForComodification();
            E result = parent.remove(parentOffset + index);
            this.modCount = parent.modCount;
            this.size--;
            //返回最终结果的元素。
            return result;
        }
```
##### remove(Object)方法
```
//通过元素来删除该元素，就依次遍历，如果有这个元素，就将该元素的索引传给fastRemobe(index)，使用这个方法来删除该元素，
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
跟进fastRemove（）方法，这个方法的作用就是删除该索引的元素：

```
    private void fastRemove(int index) {
        modCount++;
        //判断需要移动的元素个数
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        //把数组最后一位赋为null
        elementData[--size] = null; // clear to let GC do its work
    }
```
#### 1.2、LinkedList
　LinkedList是一种可以在任何位置进行高效地插入和移除操作的有序序列，它是基于双向链表实现的。
　
- LinkedList的父类：

继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。

LinkedList的接口：

- List 接口：能对它进行队列操作。
- Deque 接口：即能将LinkedList当作双端队列使用。
- Cloneable接口：即覆盖了函数clone()，能克隆。
- java.io.Serializable接口：这意味着LinkedList支持序列化，能通过序列化去传输。

先来看看节点的创建和构造方法吧：

```
  //实现Serilizable接口时，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。
  
  //当前列表的元素个数
       transient int size = 0;
       //指向首节点
      transient Node<E> first;
       //指向最后一个节点
       transient Node<E> last;
       //构建一个空列表
      public LinkedList() {
      }
      //构建一个包含集合c的列表
      public LinkedList(Collection<? extends E> c) {
          this();
        //将c中的元素都添加到此列表中
        addAll(c);
      }
```
进入addAll（）方法：

```
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);//此时 size == 0
}
```
进入另一个addAll（）方法：

```
public boolean addAll(int index, Collection<? extends E> c) {
    // 检查位置是否合法 位置是[0,size]，注意是闭区间 否则报异常
    checkPositionIndex(index);
    Object[] a = c.toArray();// 得到一个元素数组
    int numNew = a.length;// c中元素的数量
    if (numNew == 0)
        return false;// 没有元素，添加失败

    // 主要功能是找到第size个元素的前驱和后继。得到此元素需要分情况讨论。
    
    Node<E> pred, succ;// 前驱与后继
    if (index == size) {
        succ = null;// 无后继
        pred = last;// 前驱为last，即第size个元素(最后一个元素)
    } else {// 若与size不同，即index位于[0, size)之间
        succ = node(index);// 后继为第index个元素
        pred = succ.prev;// 前驱为后继的前驱
    }// 后文有详细的图片说明
    // 开始逐个插入
    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        // 新建一个以pred为前驱、null为后继、值为e的节点
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)// 前驱为空，则此节点被当做列表的第一个节点
            first = newNode;
        else// 规避掉了NullPointerException，感觉又达到了目的，又实现了逻辑
            pred.next = newNode;// 不为空，则将前驱的后继改成当前节点
        pred = newNode;// 将前驱改成当前节点，以便后续添加c中其它的元素
    }
    // 至此，c中元素已添加到链表上，但链表中从size开始的那些元素还没有链接到列表上
    // 此时就需要利用到之前找出来的succ值，它是作为这个c的整体后继
    if (succ == null) {// 如果后继为空，说明无整体后继
        last = pred;// c的最后一个元素应当作为列表的尾元素
    } else {// 有整体后继
        pred.next = succ;// pred即c中的最后一个元素，其后继指向succ，即整体后继
        succ.prev = pred;// succ的前驱指向c中的最后一个元素
    }
    // 添加完毕，修改参数
    size += numNew;
    modCount++;
    return true;
}
```
返回序号为index的元素节点：

```
Node<E> node(int index) {
    // assert isElementIndex(index);
    // 这里就和折半查找算法是类似的了，从中间分段
    if (index < (size >> 1)) {
        Node<E> x = first;
        // 循环index次 迭代到所需要的元素
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        // 循环size-1-index次
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```
插入最后一个元素e：

```
void linkLast(E e) {
    final Node<E> l = last;// 找到最后一个节点
    // 前驱为前last，值为e，后继为null
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;// last一定会指向此节点
    if (l == null)// 最后一个节点为空，说明列表中无元素
        first = newNode;// first同样指向此节点
    else
        l.next = newNode;// 否则，前last的后继指向当前节点
    size++;
    modCount++;
}
```
插入第一个元素e到succ前面：

```
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    final Node<E> pred = succ.prev; // 找到succ的前驱
    // 前驱为pred，值为e，后继为succ
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 将succ的前驱指向当前节点
    succ.prev = newNode;
    if (pred == null)// pred为空，说明此时succ为首节点
        first = newNode;// 指向当前节点
    else
        pred.next = newNode;// 否则，将succ之前的前驱的后继指向当前节点
    size++;
    modCount++;
}
```
看看add（）方法：

```
public boolean add(E e) {
     linkLast(e);
     return true;
 }
 // 只有这个是有一点逻辑的
 public void add(int index, E element) {
     checkPositionIndex(index);
     if (index == size)// 为最后一个节点，当然是添加到最后一个~
         linkLast(element);
     else
         linkBefore(element, node(index));
 }
 public void addFirst(E e) {
     linkFirst(e);
 }
 public void addLast(E e) {
     linkLast(e);
 }
```
看看删除吧：

```
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;别忽略这里的断言
    final E element = f.item;// 取出首节点中的元素
    final Node<E> next = f.next;// 取出首节点中的后继
    f.item = null;
    f.next = null; // help GC
    first = next;// first指向前first的后继，也就是列表中的2号位
    if (next == null)// 如果此时2号位为空，那么列表中此时已无节点
        last = null;// last指向null
    else
        next.prev = null;// 首节点无前驱
    size--;
    modCount++;
    return element;// 返回首节点保存的元素值
}
```
删除尾节点：

```
private E unlinkLast(Node<E> l) {
// assert l == last && l != null;别忽略这里的断言
final E element = l.item;// 取出尾节点中的元素
final Node<E> prev = l.prev;// 取出尾节点中的后继
l.item = null;
l.prev = null; // help GC
last = prev;// last指向前last的前驱，也就是列表中的倒数2号位
if (prev == null)// 如果此时倒数2号位为空，那么列表中已无节点
    first = null;// first指向null
else
    prev.next = null;// 尾节点无后继
size--;
modCount++;
return element;// 返回尾节点保存的元素值
}
```
删除某个非空节点：

```
// x即为要删除的节点
E unlink(Node<E> x) {
// assert x != null;
final E element = x.item;// 保存x的元素值
final Node<E> next = x.next;// 保存x的后继
final Node<E> prev = x.prev;// 保存x的前驱

if (prev == null) {// 前驱为null，说明x为首节点
    first = next;// first指向x的后继
} else {
    prev.next = next;// x的前驱的后继指向x的后继，即略过了x
    x.prev = null;// x.prev已无用处，置空引用
}

if (next == null) {// 后继为null，说明x为尾节点
    last = prev;// last指向x的前驱
} else {
    next.prev = prev;// x的后继的前驱指向x的前驱，即略过了x
    x.next = null;// x.next已无用处，置空引用
}

x.item = null;// 引用置空
size--;
modCount++;
return element;// 返回所删除的节点的元素值
}
```
再来看看删除的基本操作：

```
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
// 遍历列表中所有的节点，找到相同的元素，然后删除它
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
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```








