- [二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)（Facebook 在半年内面试常考）

  ```java
  class Solution {
      public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
          if(root==null||root==p||root==q) {
              return root;
          }
  
          TreeNode left=lowestCommonAncestor(root.left,p,q);
          TreeNode right=lowestCommonAncestor(root.right,p,q);
  
          if(left==null) return right;
  
          if(right==null) return left;
  
          return root;
      }
  }
  ```

  

- [从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)（字节跳动、亚马逊、微软在半年内面试中考过）

  ```java
  public TreeNode buildTree(int[] preorder, int[] inorder) {
      if (preorder.length == 0) {
          return null;
      }
      Stack<TreeNode> roots = new Stack<TreeNode>();
      int pre = 0;
      int in = 0;
      //先序遍历第一个值作为根节点
      TreeNode curRoot = new TreeNode(preorder[pre]);
      TreeNode root = curRoot;
      roots.push(curRoot);
      pre++;
      //遍历前序遍历的数组
      while (pre < preorder.length) {
          //出现了当前节点的值和中序遍历数组的值相等，寻找是谁的右子树
          if (curRoot.val == inorder[in]) {
              //每次进行出栈，实现倒着遍历
              while (!roots.isEmpty() && roots.peek().val == inorder[in]) {
                  curRoot = roots.peek();
                  roots.pop();
                  in++;
              }
              //设为当前的右孩子
              curRoot.right = new TreeNode(preorder[pre]);
              //更新 curRoot
              curRoot = curRoot.right;
              roots.push(curRoot);
              pre++;
          } else {
              //否则的话就一直作为左子树
              curRoot.left = new TreeNode(preorder[pre]);
              curRoot = curRoot.left;
              roots.push(curRoot);
              pre++;
          }
      }
      return root;
  }
  ```

  

- [组合](https://leetcode-cn.com/problems/combinations/)（微软、亚马逊、谷歌在半年内面试中考过）

  ```java
  class Solution {
     
          public List<List<Integer>> combine(int n, int k) {
  
              List<List<Integer>> res = new ArrayList<List<Integer>>();
              if (n == 0 || k == 0 || n < k) {
                  return res;
              }
  
  
              LinkedList<Integer> path = new LinkedList<Integer>();
              dfs(n, k, 1, path, res);
              return res;
          }
  
          // p 可以声明成一个栈
          // i 的极限值满足： n - i + 1 = (k - pre.size())。
          // 【关键】n - i + 1 是闭区间 [i,n] 的长度。
          // k - pre.size() 是剩下还要寻找的数的个数。
          private void dfs(int n, int k, int index, LinkedList<Integer> path, List<List<Integer>> res) {
              if (path.size() == k) {
                  res.add(new ArrayList<>(path));
                  return;
              }
  
              for (int i = index; i <= (n - (k-path.size()) + 1); i++) {
                  path.add(i);
                  dfs(n, k, i + 1, path, res);
                  path.removeLast();
              }
          }
  }
  ```

  

- [全排列](https://leetcode-cn.com/problems/permutations/)（字节跳动在半年内面试常考）

  ```java
  class Solution {
      public List<List<Integer>> permute(int[] nums) {
  
        List<List<Integer>> res=new ArrayList<>();
      
        if(nums==null || nums.length== 0) {
            return res;
        }
  
        int len=nums.length;
             
        boolean[] used=new boolean[len];
  
        LinkedList<Integer> path=new LinkedList<Integer>();
  
        dfs(nums,len,0,used,path,res); 
        
        return res;
      }
  
      private void dfs(int[] nums,int len,int depth,boolean[] used,LinkedList<Integer> path, List<List<Integer>> res) {
          if(len==depth){
              res.add(new ArrayList<>(path));
              return;
          }
  
          for(int i=0;i<len ;i++){
              if(used[i]){
                  continue;
              }
  
              used[i]=true;
  
              path.add(nums[i]);
  
              dfs(nums,len,depth+1,used,path,res);
  
              used[i]=false;
  
              path.removeLast();
          }
      }
  }
  ```

  

- [全排列 II ](https://leetcode-cn.com/problems/permutations-ii/)（亚马逊、字节跳动、Facebook 在半年内面试中考过）

  ```java
  class Solution {
     public List<List<Integer>> permuteUnique(int[] nums) {
              List<List<Integer>> res = new ArrayList<List<Integer>>();
  
              if (nums == null || nums.length == 0) {
                  return res;
              }
  
              // 排序（升序或者降序都可以），排序是剪枝的前提
              Arrays.sort(nums);
  
              int len = nums.length;
  
              boolean[] used = new boolean[len];
  
              LinkedList<Integer> path = new LinkedList<Integer>();
  
              dsf(nums, len, 0, used, path, res);
  
              return res;
          }
  
          public void dsf(int[] nums, int len, int depth, boolean[] used, LinkedList<Integer> path, List<List<Integer>> res) {
              if (len == depth) {
                  res.add(new ArrayList<Integer>(path));
                  return;
              }
  
              for (int i = 0; i < len; i++) {
  
                  if (used[i]) {
                      continue;
                  }
                  // 减树枝
                  if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
                      continue;
                  }
  
                  used[i]=true;
                  path.add(nums[i]);
                  dsf(nums, len, depth + 1, used, path, res);
  
                  path.removeLast();
                  used[i] = false;
              }
          }
  }
  ```

  