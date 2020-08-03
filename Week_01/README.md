学习笔记

##### 1.数组、链表、跳表的基本实现和特性

- [Java 源码分析（ArrayList）](http://developer.classpath.org/doc/java/util/ArrayList-source.html)
- [Linked List 的标准实现代码](http://www.geeksforgeeks.org/implementing-a-linked-list-in-java-using-class/)
- [Linked List 示例代码](http://www.cs.cmu.edu/~adamchik/15-121/lectures/Linked Lists/code/LinkedList.java)
- [Java 源码分析（LinkedList）](http://developer.classpath.org/doc/java/util/LinkedList-source.html)
- LRU Cache - Linked list：[ LRU 缓存机制](http://leetcode-cn.com/problems/lru-cache)
- Redis - Skip List：[跳跃表](http://redisbook.readthedocs.io/en/latest/internal-datastruct/skiplist.html)、[为啥 Redis 使用跳表（Skip List）而不是使用 Red-Black？](http://www.zhihu.com/question/20202931)

##### 2.栈和队列的实现与特性

队列：Queue 先进先出 http://fuseyism.com/classpath/doc/java/util/Queue-source.html

栈：Stack 先进后出 http://developer.classpath.org/doc/java/util/Stack-source.html

双端队列：Deque https://www.cnblogs.com/fzxey/p/10793147.html

优先队列 https://docs.oracle.com/javase/10/docs/api/java/util/PriorityQueue.html

1.插入操作：O(1)

2.取出操作：O(logn)-按照元素的优先级取出

3.底层具体实现的数据结构较为多样和复杂：heap

