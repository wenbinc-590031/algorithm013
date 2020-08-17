##### 5.递归

本质上就是循环，调用自己

效率其实没有那么差。

​           优化的方式：1.缓存其中的值

​                                   2.尾递归：原理不需要新建一个栈，覆盖原来的栈

```java
// Java
public void recur(int level, int param) { 
  // terminator 
  if (level > MAX_LEVEL) { 
    // process result 
    return; 
  }
  // process current logic 
  process(level, param); 
  // drill down 
  recur( level: level + 1, newParam); 
  // restore current status 
 
}
```



三个思维要点：

1.不要人肉进行递归

2.找到最近最简单的方法。将其拆解成可重复解决的问题

3.数学归纳思维

###### 如何优雅地计算斐波那契数列？

​    f(n)=f(n-1)+f(n-2)

   1.递归

2. 动态规划
3. 通用公式



- [分治代码模板](https://shimo.im/docs/zvlDqLLMFvcAF79A/)

```java
private static int divide_conquer(Problem problem, ) {
  
  if (problem == NULL) {
    int res = process_last_result();
    return res;     
  }
  subProblems = split_problem(problem)
  
  res0 = divide_conquer(subProblems[0])
  res1 = divide_conquer(subProblems[1])
  
  result = process_result(res0, res1);
  
  return result;
}
```

