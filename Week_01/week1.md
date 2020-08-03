### 简单：

- 用 add first 或 add last 这套新的 API 改写 Deque 的代码

  ```java
          Deque<String> deque=new LinkedList<String>();
          deque.addFirst("a");
          deque.addFirst("b");
          deque.addFirst("c");
  
          System.out.println(deque);
  
          String str2=deque.peekFirst();
          System.out.println(str2);
          System.out.println(deque);
  
  
          while (deque.size()>0){
              System.out.println(deque.pollFirst());
          }
  ```

  

- 分析 Queue 和 Priority Queue 的源码

  ```java
  public interface Queue<E> extends Collection<E> {
      // 添加元素，成功返回true，失败报异常
      boolean add(E e);
      // 添加元素，成功返回true，失败返回false
      boolean offer(E e);
      // 取出并删除对头，如果队列空，则报异常
      E remove();
      // 取出并删除对头，如果队列空，则返回null
      E poll();
      // 取出不删除对头，如果队列空，则报异常
      E element();
      // 取出不删除对头，如果队列空，则返回null
      E peek();
  }
  ```

  ```java
  /**
  Priority Queue 优先级队列
  
  插入操作O（1）；取出操作O(logN)按优先级取出
  底层数据结构多样，可以是heap、二叉树、Fibonacci堆、红黑树、treap
  优先级排序实现Comparator接口
  **/
  
  package com.example.demo;
  
  import java.util.*;
  import java.util.function.Consumer;
  
  public class PriorityQueue<E> extends AbstractQueue<E> implements java.io.Serializable {
  
      // 默认大小
      private static final int DEFAULT_INITIAL_CAPACITY = 11;
      // 队列。底层是平衡二叉树。按照自定义序或者自然序。不允许空。
      // 只保证了头元素是最小的。并不保证有序。
      transient Object[] queue; // transient关键字标识不会被序列化
      // 队列元素个数
      private int size = 0;
      // 自定义比较器。空就使用自然序。
      private final Comparator<? super E> comparator;
      // 队列被结构性修改的次数。
      transient int modCount = 0;
      // 创建初始值为11的自然序队列
      public PriorityQueue() {
          this(DEFAULT_INITIAL_CAPACITY, null);
      }
      // 指定初始值大小
      public PriorityQueue(int initialCapacity) {
          this(initialCapacity, null);
      }
      public PriorityQueue(Comparator<? super E> comparator) {
          this(DEFAULT_INITIAL_CAPACITY, comparator);
      }
      public PriorityQueue(int initialCapacity,
                           Comparator<? super E> comparator) {
          if (initialCapacity < 1)
              throw new IllegalArgumentException();
          this.queue = new Object[initialCapacity];
          this.comparator = comparator;
      }
      public PriorityQueue(Collection<? extends E> c) {
          // 是SortedSet实例，就用SortedSet的比较器
          if (c instanceof SortedSet<?>) {
              SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
              this.comparator = (Comparator<? super E>) ss.comparator();
              initElementsFromCollection(ss);
          }
          // 是PriorityQueue实例，就用PriorityQueue的比较器
          else if (c instanceof PriorityQueue<?>) {
              PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
              this.comparator = (Comparator<? super E>) pq.comparator();
              initFromPriorityQueue(pq);
          }
          else {
              // 其余情况，没有自定义比较器
              this.comparator = null;
              initFromCollection(c);
          }
      }
      public PriorityQueue(PriorityQueue<? extends E> c) {
          this.comparator = (Comparator<? super E>) c.comparator();
          initFromPriorityQueue(c);
      }
      public PriorityQueue(SortedSet<? extends E> c) {
          this.comparator = (Comparator<? super E>) c.comparator();
          initElementsFromCollection(c);
      }
      private void initFromPriorityQueue(PriorityQueue<? extends E> c) {
          if (c.getClass() == PriorityQueue.class) {
              // 队列转成数组
              this.queue = c.toArray();
              this.size = c.size();
          } else {
              initFromCollection(c);
          }
      }
  
      private void initElementsFromCollection(Collection<? extends E> c) {
          Object[] a = c.toArray();
          // If c.toArray incorrectly doesn't return Object[], copy it.
          if (a.getClass() != Object[].class)
              a = Arrays.copyOf(a, a.length, Object[].class);
          int len = a.length;
          if (len == 1 || this.comparator != null)
              for (int i = 0; i < len; i++)
                  if (a[i] == null)
                      throw new NullPointerException();
          this.queue = a;
          this.size = a.length;
      }
      // 根据给定集合初始值队列数组
      private void initFromCollection(Collection<? extends E> c) {
          initElementsFromCollection(c);
          // 构建最大堆
          heapify();
      }
      // 虚拟机能开辟的最大数组
      private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
      // 数组扩容
      // 时间复杂度O(logN)
      private void grow(int minCapacity) {
          int oldCapacity = queue.length;
          // 不足64。扩容两倍+2。超过64，扩容1.5倍。
          int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                           (oldCapacity + 2) :
                                           (oldCapacity >> 1));
          // overflow-conscious code
          if (newCapacity - MAX_ARRAY_SIZE > 0)
              newCapacity = hugeCapacity(minCapacity);
          // 将原来的是数组拷贝到新数组
          queue = Arrays.copyOf(queue, newCapacity);
      }
  
      private static int hugeCapacity(int minCapacity) {
          if (minCapacity < 0) // 溢出，报OOM错误
              throw new OutOfMemoryError();
          return (minCapacity > MAX_ARRAY_SIZE) ?
              Integer.MAX_VALUE :
              MAX_ARRAY_SIZE;
      }
      // add底层调的offer
      public boolean add(E e) {
          return offer(e);
      }
      // 判断数组是否需要扩容，需要则调用grow犯法
      // 插入元素后，可能破坏堆的平衡，调用siftUp（上滤）方法调整堆至平衡
      public boolean offer(E e) {
          // 添加元素为空，抛异常
          if (e == null)
              throw new NullPointerException();
          // 修改数+1
          modCount++;
          int i = size;
          // 个数超过数组大小，扩容
          if (i >= queue.length)
              grow(i + 1);
          size = i + 1;
          if (i == 0) // 空数组直接加
              queue[0] = e;
          else
              siftUp(i, e); // i数组大小，e添加元素
          return true;
      }
  
      public E peek() {
          return (size == 0) ? null : (E) queue[0];
      }
  
      private int indexOf(Object o) {
          if (o != null) {
              for (int i = 0; i < size; i++)
                  if (o.equals(queue[i]))
                      return i;
          }
          return -1;
      }
  
      public boolean remove(Object o) {
          int i = indexOf(o);
          if (i == -1)
              return false;
          else {
              removeAt(i);
              return true;
          }
      }
  
      boolean removeEq(Object o) {
          for (int i = 0; i < size; i++) {
              if (o == queue[i]) {
                  removeAt(i);
                  return true;
              }
          }
          return false;
      }
  
      public boolean contains(Object o) {
          return indexOf(o) != -1;
      }
  
      public Object[] toArray() {
          return Arrays.copyOf(queue, size);
      }
  
      public <T> T[] toArray(T[] a) {
          final int size = this.size;
          if (a.length < size)
              // Make a new array of a's runtime type, but my contents:
              return (T[]) Arrays.copyOf(queue, size, a.getClass());
          System.arraycopy(queue, 0, a, 0, size);
          if (a.length > size)
              a[size] = null;
          return a;
      }
  
      public Iterator<E> iterator() {
          return new Itr();
      }
      // 普通数组的遍历
      // 内部类Itr，利用curosr指针迭代内部数组queue
      private final class Itr implements Iterator<E> {
  
          private int cursor = 0;
  
          private int lastRet = -1;
  
          private ArrayDeque<E> forgetMeNot = null;
  
          private E lastRetElt = null;
  
          private int expectedModCount = modCount;
  
          public boolean hasNext() {
              return cursor < size ||
                  (forgetMeNot != null && !forgetMeNot.isEmpty());
          }
  
          public E next() {
              if (expectedModCount != modCount)
                  throw new ConcurrentModificationException();
              if (cursor < size)
                  return (E) queue[lastRet = cursor++];
              if (forgetMeNot != null) {
                  lastRet = -1;
                  lastRetElt = forgetMeNot.poll();
                  if (lastRetElt != null)
                      return lastRetElt;
              }
              throw new NoSuchElementException();
          }
  
          public void remove() {
              if (expectedModCount != modCount)
                  throw new ConcurrentModificationException();
              if (lastRet != -1) {
                  E moved = PriorityQueue.this.removeAt(lastRet);
                  lastRet = -1;
                  if (moved == null)
                      cursor--;
                  else {
                      if (forgetMeNot == null)
                          forgetMeNot = new ArrayDeque<>();
                      forgetMeNot.add(moved);
                  }
              } else if (lastRetElt != null) {
                  PriorityQueue.this.removeEq(lastRetElt);
                  lastRetElt = null;
              } else {
                  throw new IllegalStateException();
              }
              expectedModCount = modCount;
          }
      }
  
      public int size() {
          return size;
      }
  
      public void clear() {
          modCount++;
          for (int i = 0; i < size; i++)
              queue[i] = null;
          size = 0;
      }
  
      // 删除
      public E poll() {
          if (size == 0)
              return null;
          int s = --size;
          modCount++;
          E result = (E) queue[0];
          E x = (E) queue[s];
          queue[s] = null;
          if (s != 0)
              siftDown(0, x);
          return result;
      }
  
      private E removeAt(int i) {
          // assert i >= 0 && i < size;
          modCount++;
          int s = --size;
          if (s == i) // removed last element
              queue[i] = null;
          else {
              E moved = (E) queue[s];
              queue[s] = null;
              // 与下层元素交换
              siftDown(i, moved);
              if (queue[i] == moved) {
                  siftUp(i, moved);
                  if (queue[i] != moved)
                      return moved;
              }
          }
          return null;
      }
      // 自底向上，不断比较、交换
      private void siftUp(int k, E x) {
          if (comparator != null) // 用自定义比较器
              siftUpUsingComparator(k, x);
          else
              siftUpComparable(k, x); // 用自然序比较器
      }
      // 在数组k的位置插入x
      // 把x赋值给key
      // 不断比较key和k的父节点e（queue[(k-1)/2]）的关系
      // 1、若key<e，则queue[k]=e，k回溯至e
      // 2、若key>=e捉着k已经到达根节点，则结束循环
      // queue[k]=e
      // 时间复杂度O(logn)
      private void siftUpComparable(int k, E x) { // k数组大小，x添加元素
          Comparable<? super E> key = (Comparable<? super E>) x; // 把x赋值给key
          while (k > 0) { // 当k不是根节点
              int parent = (k - 1) >>> 1; // 取出k父节点parent下标
              Object e = queue[parent]; // 取出parent值
              if (key.compareTo((E) e) >= 0) // 比较key>=父节点e。结束循环
                  break;
              queue[k] = e; // 赋值操作
              k = parent; // k指针回溯操作
          }
          queue[k] = key;
      }
  
      private void siftUpUsingComparator(int k, E x) {
          while (k > 0) {
              int parent = (k - 1) >>> 1;
              Object e = queue[parent];
              if (comparator.compare(x, (E) e) >= 0)
                  break;
              queue[k] = e;
              k = parent;
          }
          queue[k] = x;
      }
  
      private void siftDown(int k, E x) {
          if (comparator != null)
              siftDownUsingComparator(k, x);
          else
              siftDownComparable(k, x);
      }
  
      // 自顶向下
      // 时间复杂度O(logn)
      private void siftDownComparable(int k, E x) {
          Comparable<? super E> key = (Comparable<? super E>)x; // x赋值给key
          int half = size >>> 1;        // 当不是叶子节点时执行循环
          while (k < half) {
              int child = (k << 1) + 1; // 左孩子
              Object c = queue[child]; // 左孩子的值
              int right = child + 1; // 右孩子
              if (right < size &&
                  ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0) // 左孩子大于右孩子的情况
                  c = queue[child = right]; // right赋值给child,c的值变成右孩子的值
              if (key.compareTo((E) c) <= 0) // 比较key和c的大小
                  break;
              queue[k] = c;
              k = child;
          }
          queue[k] = key;
      }
  
      private void siftDownUsingComparator(int k, E x) {
          int half = size >>> 1;
          while (k < half) {
              int child = (k << 1) + 1;
              Object c = queue[child];
              int right = child + 1;
              if (right < size &&
                  comparator.compare((E) c, (E) queue[right]) > 0)
                  c = queue[child = right];
              if (comparator.compare(x, (E) c) <= 0)
                  break;
              queue[k] = c;
              k = child;
          }
          queue[k] = x;
      }
      // 构建最大堆
      // 把数组看一个未调整完毕的堆
      // 从第一个非叶子节点开始，从下往上、从右往左不断调用siftDown方法
      private void heapify() {
          for (int i = (size >>> 1) - 1; i >= 0; i--)
              siftDown(i, (E) queue[i]);
      }
  
      public Comparator<? super E> comparator() {
          return comparator;
      }
  
      private void writeObject(java.io.ObjectOutputStream s)
          throws java.io.IOException {
          // Write out element count, and any hidden stuff
          s.defaultWriteObject();
  
          // Write out array length, for compatibility with 1.5 version
          s.writeInt(Math.max(2, size + 1));
  
          // Write out all elements in the "proper order".
          for (int i = 0; i < size; i++)
              s.writeObject(queue[i]);
      }
  
      private void readObject(java.io.ObjectInputStream s)
          throws java.io.IOException, ClassNotFoundException {
          // Read in size, and any hidden stuff
          s.defaultReadObject();
  
          // Read in (and discard) array length
          s.readInt();
  
          queue = new Object[size];
  
          // Read in all elements.
          for (int i = 0; i < size; i++)
              queue[i] = s.readObject();
  
          // Elements are guaranteed to be in "proper order", but the
          // spec has never explained what that might be.
          heapify();
      }
  
      public final Spliterator<E> spliterator() {
          return new PriorityQueueSpliterator<E>(this, 0, -1, 0);
      }
  
      static final class PriorityQueueSpliterator<E> implements Spliterator<E> {
          /*
           * This is very similar to ArrayList Spliterator, except for
           * extra null checks.
           */
          private final PriorityQueue<E> pq;
          private int index;            // current index, modified on advance/split
          private int fence;            // -1 until first use
          private int expectedModCount; // initialized when fence set
  
          /** Creates new spliterator covering the given range */
          PriorityQueueSpliterator(PriorityQueue<E> pq, int origin, int fence,
                               int expectedModCount) {
              this.pq = pq;
              this.index = origin;
              this.fence = fence;
              this.expectedModCount = expectedModCount;
          }
  
          private int getFence() { // initialize fence to size on first use
              int hi;
              if ((hi = fence) < 0) {
                  expectedModCount = pq.modCount;
                  hi = fence = pq.size;
              }
              return hi;
          }
  
          public PriorityQueueSpliterator<E> trySplit() {
              int hi = getFence(), lo = index, mid = (lo + hi) >>> 1;
              return (lo >= mid) ? null :
                  new PriorityQueueSpliterator<E>(pq, lo, index = mid,
                                                  expectedModCount);
          }
  
          public void forEachRemaining(Consumer<? super E> action) {
              int i, hi, mc; // hoist accesses and checks from loop
              PriorityQueue<E> q; Object[] a;
              if (action == null)
                  throw new NullPointerException();
              if ((q = pq) != null && (a = q.queue) != null) {
                  if ((hi = fence) < 0) {
                      mc = q.modCount;
                      hi = q.size;
                  }
                  else
                      mc = expectedModCount;
                  if ((i = index) >= 0 && (index = hi) <= a.length) {
                      for (E e;; ++i) {
                          if (i < hi) {
                              if ((e = (E) a[i]) == null) // must be CME
                                  break;
                              action.accept(e);
                          }
                          else if (q.modCount != mc)
                              break;
                          else
                              return;
                      }
                  }
              }
              throw new ConcurrentModificationException();
          }
  
          public boolean tryAdvance(Consumer<? super E> action) {
              if (action == null)
                  throw new NullPointerException();
              int hi = getFence(), lo = index;
              if (lo >= 0 && lo < hi) {
                  index = lo + 1;
                  @SuppressWarnings("unchecked") E e = (E)pq.queue[lo];
                  if (e == null)
                      throw new ConcurrentModificationException();
                  action.accept(e);
                  if (pq.modCount != expectedModCount)
                      throw new ConcurrentModificationException();
                  return true;
              }
              return false;
          }
  
          public long estimateSize() {
              return (long) (getFence() - index);
          }
  
          public int characteristics() {
              return Spliterator.SIZED | Spliterator.SUBSIZED | Spliterator.NONNULL;
          }
      }
  }
  
  ```

  

- [删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array/)（Facebook、字节跳动、微软在半年内面试中考过）

  ```java
  class Solution {
      public int removeDuplicates(int[] nums) {
         if(nums==null){
             return 0;
         }
  
         if(nums.length==1){
             return 1;
         }
         int j=0;
  
         for(int i=1;i<nums.length;i++){
             if(nums[i]!=nums[j]){
                 j++;
                 nums[j]=nums[i];
             }
         }
  
         return j+1;
      }
  }
  ```

  

- [旋转数组](https://leetcode-cn.com/problems/rotate-array/)（微软、亚马逊、PayPal 在半年内面试中考过）

  ```java
  public class Solution {
      public void rotate(int[] nums, int k) {
          k = k % nums.length;
          int count = 0;
          for (int start = 0; count < nums.length; start++) {
              int current = start;
              int prev = nums[start];
              do {
                  int next = (current + k) % nums.length;
                  int temp = nums[next];
                  nums[next] = prev;
                  prev = temp;
                  current = next;
                  count++;
              } while (start != current);
          }
      }
  }
  ```

  

- [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/)（亚马逊、字节跳动在半年内面试常考）

  ```java
  class Solution {
      public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
  
          ListNode pre=new ListNode(0);
          ListNode temp=pre;
  
          while(l1!=null&&l2!=null){
              if(l1.val<l2.val){
                  temp.next=l1;
                  l1=l1.next;
              }else{
                  temp.next=l2;
                  l2=l2.next;
              }
              temp=temp.next;
          }
  
          temp.next=l1==null?l2:l1;
          return pre.next;
      }
  }
  ```

  

- [合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array/)（Facebook 在半年内面试常考）

  ```java
  class Solution {
      public void merge(int[] nums1, int m, int[] nums2, int n) {
          int len1=m-1;
          int len2=n-1;
          int len=m+n-1;
  
          while(len1>=0&&len2>=0){
              nums1[len--]=nums1[len1]>nums2[len2]?nums1[len1--]:nums2[len2--];
          }
  
          //拷贝
  
          System.arraycopy(nums2,0,nums1,0,len2+1);
      }
  }
  ```

  

- [两数之和](https://leetcode-cn.com/problems/two-sum/)（亚马逊、字节跳动、谷歌、Facebook、苹果、微软在半年内面试中高频常考）

  ```java
  /*
  标签：哈希映射
  这道题本身如果通过暴力遍历的话也是很容易解决的，时间复杂度在 O(n2)
  由于哈希查找的时间复杂度为 O(1)，所以可以利用哈希容器 map 降低时间复杂度
  遍历数组 nums，i 为当前下标，每个值都判断map中是否存在 target-nums[i] 的 key 值
  如果存在则找到了两个值，如果不存在则将当前的 (nums[i],i) 存入 map 中，继续遍历直到找到为止
  如果最终都没有结果则抛出异常
  时间复杂度：O(n)
  */
  class Solution {
      public int[] twoSum(int[] nums, int target) {
  
          Map<Integer,Integer> map=new HashMap();
          for(int i=0;i<nums.length;i++){
             int value=target-nums[i];
             if(map.containsKey(value)){
                 return new int[]{map.get(value),i};
             }
             map.put(nums[i],i);
          }
  
          return null;
      }
  }
  ```

  

- [移动零](https://leetcode-cn.com/problems/move-zeroes/)（Facebook、亚马逊、苹果在半年内面试中考过）

  ```java
  class Solution {
      public void moveZeroes(int[] nums) {
         int j=0;
  
         for(int i=0; i<nums.length;i++) {
             if(nums[i]!=0){
                 if(i!=j){
                   int temp=nums[i];
                   nums[i]=nums[j];
                   nums[j]=temp;
                 }
                 j++;
             }
         }
      }
  }
  ```

  

- [加一](https://leetcode-cn.com/problems/plus-one/)（谷歌、字节跳动、Facebook 在半年内面试中考过）

  ```java
  class Solution {
      public int[] plusOne(int[] digits) {
        if (digits == null) {
                  return digits;
              }
  
              int len = digits.length;
  
              for (int i = len-1; i >= 0; i--) {
                  if (digits[i] < 9) {
                      digits[i]++;
                      return digits;
                  }
  
                  digits[i] = 0;
              }
  
              digits = new int[len + 1];
              digits[0] = 1;
              return digits;
      }
  }
  ```

### 中等：

- [设计循环双端队列](https://leetcode.com/problems/design-circular-deque)（Facebook 在 1 年内面试中考过）

  ```java
  class MyCircularDeque {
  
       // 长度
      private int capacity;
      //数组
      private int[] array;
      // 头指针
      private int front;
      //尾指针
      private int rear;
  
  
      /**
       * Initialize your data structure here. Set the size of the deque to be k.
       */
      public MyCircularDeque(int k) {
          capacity = k + 1;
          array = new int[capacity];
          front = 0;
          rear = 0;
      }
  
      /**
       * Adds an item at the front of Deque. Return true if the operation is successful.
       */
      public boolean insertFront(int value) {
          if (isFull()) {
              return false;
          }
  
          front = (front + capacity - 1) % capacity;
          array[front] = value;
          return true;
      }
  
      /**
       * Adds an item at the rear of Deque. Return true if the operation is successful.
       */
      public boolean insertLast(int value) {
          if (isFull()) {
              return false;
          }
  
          array[rear] = value;
          rear = (rear + 1) % capacity;
          return true;
      }
  
      /**
       * Deletes an item from the front of Deque. Return true if the operation is successful.
       */
      public boolean deleteFront() {
          if (isEmpty()) {
              return false;
          }
  
          front = (front + 1) % capacity;
          return true;
      }
  
      /**
       * Deletes an item from the rear of Deque. Return true if the operation is successful.
       */
      public boolean deleteLast() {
  
          if (isEmpty()) {
              return false;
          }
  
          rear = (rear + capacity - 1) % capacity;
          return true;
      }
  
      /**
       * Get the front item from the deque.
       */
      public int getFront() {
          if (isEmpty()) {
              return -1;
          }
          return array[front];
      }
  
      /**
       * Get the last item from the deque.
       */
      public int getRear() {
          if (isEmpty()) {
              return -1;
          }
          return array[(rear+capacity-1)%capacity];
      }
  
      /**
       * Checks whether the circular deque is empty or not.
       */
      public boolean isEmpty() {
          //如果头指针==尾指针。则代表weinull
          if (front == rear) {
              return true;
          }
          return false;
      }
  
      /**
       * Checks whether the circular deque is full or not.
       */
      public boolean isFull() {
          //如果尾指针在头指针的前面。则代表满了
          if ((rear + 1) % capacity == front) {
              return true;
          }
          return false;
      }
  }
  
  ```

  

### 困难：

- [接雨水](https://leetcode.com/problems/trapping-rain-water/)（亚马逊、字节跳动、高盛集团、Facebook 在半年内面试常考）

