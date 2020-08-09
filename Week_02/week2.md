### 简单：

- 写一个关于 HashMap 的小总结。

- [有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram/description/)

  ```java
  //1.Sort 排序
  class Solution {
      public boolean isAnagram(String s, String t) {
            if (s.length() != t.length()) {
              return false;
          }
  
          char[] str1 = s.toCharArray();
          char[] str2 = t.toCharArray();
          Arrays.sort(str1);
          Arrays.sort(str2);
  
          return Arrays.equals(str1,str2);
      }
  }
  
  //2. 通过数组的形式
  class Solution {
      public boolean isAnagram(String s, String t) {
          int [] map = new int [26];
          for (char c: s.toCharArray()) {
              map[c - 'a'] ++;
          }
          for (char c: t.toCharArray()) {
              map[c - 'a'] --;
          }
          for (int n: map) {
              if (n != 0) {
                  return false;
              }
          }
          return true;
      }
  }
  ```

- [字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)（亚马逊在半年内面试中常考）

  ```java
  class Solution {
      public List<List<String>> groupAnagrams(String[] strs) {
  
          if (strs == null || strs.length == 0) {
              return new ArrayList<List<String>>();
          }
  
          Map<String, List<String>> map = new HashMap();
  
          for (int i = 0; i < strs.length; i++) {
  
              char[] ca = strs[i].toCharArray();
  
              Arrays.sort(ca);
              String keySet = String.valueOf(ca);
              if (!map.containsKey(keySet)) {
                  map.put(keySet, new ArrayList<String>());
              }
  
              map.get(keySet).add(keySet);
          }
  
          return new ArrayList<List<String>>(map.values());
  
      }
  }
  ```

  

- [两数之和](https://leetcode-cn.com/problems/two-sum/description/)（近半年内，亚马逊考查此题达到 216 次、字节跳动 147 次、谷歌 104 次，Facebook、苹果、微软、腾讯也在近半年内面试常考）

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
              int difference=target-nums[i];
  
              if(map.containsKey(difference)){
                  return new int[]{map.get(difference),i};
              }
              map.put(nums[i],i);
          }
          throw new IllegalArgumentException("No two sum solution");
      }
  }
  ```

  

- [N 叉树的前序遍历](https://leetcode-cn.com/problems/n-ary-tree-preorder-traversal/description/)（亚马逊在半年内面试中考过）

  ```java
  // 递归 快
  class Solution {
      public List<Integer> preorder(Node root) {
          if (root==null) {
              return new ArrayList<>();
          }  
          List<Integer> list=new ArrayList<>();
          helper(root,list);
          return list;
      }
  
      private void helper(Node root, List<Integer> res) {
      if (root == null) return;
      res.add(root.val);
      for (Node node : root.children) {
          helper(node, res);
      }
  }
  
  // 迭代 慢
   List<Integer> res = new ArrayList<>();
      if (root == null) return res;
      Stack<Node> stack = new Stack<>();
      stack.push(root);
      while (!stack.isEmpty()) {
          Node cur = stack.pop();
          //头结点加入结果集
          res.add(cur.val);
          //将该节点的子节点从右往左压入栈
          List<Node> nodeList = cur.children;
          for (int i = nodeList.size() - 1; i >= 0; i--) {
              stack.push(nodeList.get(i));
          }
      }
      return res;
  
  ```

  

- #### [二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

```java
class Solution {
     public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<Integer>();

        if (root == null) {
            return res;
        }

        hepler(root,res);

        return res;
    }

    public void hepler(TreeNode root,List<Integer> res){

        if(root==null){
            return;
        }

        if(root.left!=null){
            hepler(root.left,res);
        }

        res.add(root.val);

        if(root.right!=null){
            hepler(root.right,res);
        }

    }
}
```

- [二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal/)（字节跳动、谷歌、腾讯在半年内面试中考过）

  ```java
  class Solution {
      public List<Integer> preorderTraversal(TreeNode root) {
          List<Integer> res = new ArrayList<>();
          helper(root, res);
          return res;
      }
  
      private void helper(TreeNode root, List<Integer> res) {
          if (root == null) return;
          res.add(root.val);
          helper(root.left, res);
          helper(root.right, res);
      }
  }
  ```

  

- [丑数](https://leetcode-cn.com/problems/chou-shu-lcof/)（字节跳动在半年内面试中考过）

  ```java
  class Solution {
      public int nthUglyNumber(int n) {
          int a = 0, b = 0, c = 0;
          int[] dp = new int[n];
          dp[0] = 1;
          for(int i = 1; i < n; i++) {
              int n2 = dp[a] * 2, n3 = dp[b] * 3, n5 = dp[c] * 5;
              dp[i] = Math.min(Math.min(n2, n3), n5);
              if(dp[i] == n2) a++;
              if(dp[i] == n3) b++;
              if(dp[i] == n5) c++;
          }
          return dp[n - 1];
      }
  }
  
  ```

