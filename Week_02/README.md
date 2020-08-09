学习笔记

##### 1.哈希表、映射、集合的实现与特性

哈希表

​       哈希表也叫散列表，是根据关键码值而进行的访问的数据结构。它通过把关键码值映射到表中的一个位置来访问记录,以加快查找的速度。

​       这个映射函数叫做散列表函数，存放的数组叫做哈希表。

哈希表实现原理：

​       假如 我们要存放一个lies字符串 如果存？我们通过哈希函数，将lies传给哈希函数之后，它就会返回一个下标，这个下标就是一个整数，这里返回的下标是个9，就把这个lies存在9的位置，像类似于这样存下来即可。哈希函数的可以有很多种，在java中，有一个叫hashcode的方法，需要我们重载一下。图中的哈希函数是，把每个数的ASCII码加在一起，再mod上一个数

​       ![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20200402174439640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc5MDYyMw==,size_16,color_FFFFFF,t_70)

​         哈希函数，尽量需要选择好一些，这样可以避免发生碰撞，因为lies计算出来为9，其他的字符串计算出来也有可能得到9，像这种情况，我们叫做哈希碰撞。对于不同的要存储的数据，它经过哈希函数之后会得到相同的值，对应的下标都是在9这个位置。如何处理？这说明9这个位置要存多个元素，我们要做的事情也很简单，就是再增加一个维度，这个位置不止存在一个数，而是存多个数，拉出来一个链表，这样的方法叫做拉链式解决冲突法。拉出一个链表，把都应该放到9这个位置的元素依次在链表中弄出来。这时候，我们放入一个数据通过哈希得到它下标的值，这整个操作是O（1）的，也就是O（1）可以查询到它的位置。但是，很多的元素积累到相同的位置，这时候，它的查询时间就要遍历链表，为O(n)的级别。所以，我们需要找一个好的哈希函数，以此，减少它碰撞的概率。所以，在一般时候，我们都认为哈希表的查询是O（1）的

​         ![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20200402215031789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzc5MDYyMw==,size_16,color_FFFFFF,t_70)

​     总结：查询没有碰撞的时候是O(1),在有碰撞的时候，使用链表的时候为O(n)

***\*优点：\**不论哈希表中有多少数据，查找、插入、删除（有时包括删除）只需要接近常量的时间即0(1）的时间级。**

***\*缺点：\**它是\**基于数组\**的，数组创建后难于扩展，某些哈希表被基本填满时，性能下降得非常严重，所以程序员\**必须要清楚表中将要存储多少数据\****



HashMap 的原理和分析

1.概述

   HashMap 是基于Map接口实现的，元素是以键值对的方式储存，允许使用null来作为key或者value， 单key不可以重复，顺序是无序的，线程不安全。

2.继承关系

​    public class HashMap<K,V> extends AbstractMap<K,V>    implements Map<K,V>, Cloneable, Serializable

3.基本属性

```java
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; //默认初始化大小 16
static final float DEFAULT_LOAD_FACTOR = 0.75f; //负载因子0.75
static final Entry<?,?>[] EMPTY_TABLE = {}; //初始化的默认数组
transient int size; //HashMap中元素的数量
int threshold; //判断是否需要调整HashMap的容量
```

HashMap 扩容操作是一项很耗时的操作，所以如果能估算Map的容量，可以给他一个初始值，避免多次扩容，线程不安全，多线程安全可以使用**ConcurrentHashMap**（使用额了分段锁）



HashMap和HashTable的区别

 1.线程安全

​      HashMap线程不安全：在并发下，可能会造成环状链表，导致get操作时，cpu空转。

​      HashTable是线程安全的但是代价比较大，get/put所有相关操作都是synchronized的，这相当于给整个哈希表加了一把大锁，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞

2、针对null的不同
      HashMap可以使用null作为key，而Hashtable则不允许null作为key
虽说HashMap支持null值作为key，不过建议还是尽量避免这样使用，因为一旦不小心使用了，若因此引发一些问题，排查起来很是费事。
**Note：HashMap以null作为key时，总是存储在table数组的第一个节点上。**

3、继承结构
      HashMap是对Map接口的实现，HashTable实现了Map接口和Dictionary抽象类。

4、初始容量与扩容
      HashMap的初始容量为16，Hashtable初始容量为11，两者的填充因子默认都是0.75。

​      HashMap扩容时是当前容量翻倍即:capacity*2，Hashtable扩容时是容量翻倍+1即:capacity*2+1。

5、两者计算hash的方法不同
      Hashtable计算hash是直接使用key的hashcode对table数组的长度直接进行取模



HashMap的数据结构

​     HashMap由数组和链表来实现对数据的存储（jdk1.7）

​     **HashMap采用数组+链表+红黑树实现**（jdk1.8）

​     **put方法简单解析**

```java
public V put(K key, V value) {
    //调用putVal()方法完成
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //判断table是否初始化，否则初始化操作
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    //计算存储的索引位置，如果没有元素，直接赋值
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        //节点若已经存在，执行赋值操作
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        //判断链表是否是红黑树
        else if (p instanceof TreeNode)
            //红黑树对象操作
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            //为链表，
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //链表长度8，将链表转化为红黑树存储
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //key存在，直接覆盖
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //记录修改次数
    ++modCount;
    //判断是否需要扩容
    if (++size > threshold)
        resize();
    //空操作
    afterNodeInsertion(evict);
    return null;
}
```

##### 2.树，二叉树，二叉搜索树，堆，二叉堆

二叉树的面试就是递归，栈（Stack）

递归：效率其实没有那么差。

​           优化的方式：1.缓存其中的值

​                                   2.尾递归：原理不需要新建一个栈，覆盖原来的栈

​                                    

```js
//没有用尾递归
function recsum(x) {
    if (x===1) {
        return x;
    } else {
        return x + recsum(x-1);
    }
}

//使用了尾递归
function tailrecsum(x, running_total=0) {
    if (x===0) {
        return running_total;
    } else {
        return tailrecsum(x-1, running_total+x);
    }
}
```

二叉树的遍历

![img](https://img2018.cnblogs.com/blog/1590962/201903/1590962-20190306170951841-1886726002.jpg)

前序遍历：根节点->左子树->右子树**（根->左->右）**

中序遍历：左子树->根节点->右子树**（左->根->右）**

后序遍历：左子树->右子树->根节点**（左->右->根）**

​       二叉搜索树：二叉排序树

​      堆：Heap 

​     可以迅速从一堆数据中找到最大或者最小的的数据结构

​      find-max ：O(1)

​      delete-max:Olog(n)

​      insert : OlogN or O(1)

​     二叉堆：是用二叉树实现的

​      根节点：a[0]

​      索引为i的左孩子的索引是 (2i+1)

​      索引为i的右孩子的索引是 (2i+2)

​      索引为i的父节点的索引是 foolr(i-1)/2