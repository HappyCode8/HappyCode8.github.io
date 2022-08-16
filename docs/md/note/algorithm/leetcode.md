

[仓库](https://github.com/github20131983/Algorithm)

## 链表

```java
public class ListNode {
    int val;
   ListNode next;
   ListNode(int x) { 
       val = x; 
   }
}
```

1. 给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。请你将两个数相加，并以相同形式返回一个表示和的链表。你可以假设除了数字 0 之外，这两个数都不会以 0 开头。
   链接：https://leetcode-cn.com/problems/add-two-numbers
   
   思路：这个链表从前到后实际上是个位、十位、百位.......，所以从前往后刚好是模拟了从个位加到最高位的过程，基本上只要模拟就行了。
   
   ```java
   public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
           ListNode sum=new ListNode(-1);
           ListNode head=sum;
           int sigleSum=0;
           while(l1!=null||l2!=null){
               sigleSum/=10;
               if(l1!=null){
                   sigleSum+=l1.val;
                   l1=l1.next;
               }
               if(l2!=null){
                   sigleSum+=l2.val;
                   l2=l2.next;
               }
               sum.next=new ListNode(sigleSum%10);
               sum=sum.next;
           }
           if(sigleSum>=10)sum.next=new ListNode(1);
           return head.next;
       }
   ```

2. 给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。
   
   **进阶：**你能尝试使用一趟扫描实现吗？
   
   链接：https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/
   
   思路：双指针，快的先跑n个，慢的在开始跑，当快的跑到末尾时，慢的指的就是要删除的
   
   ```java
   class Solution {
       public ListNode removeNthFromEnd(ListNode head, int n) {
           ListNode start=new ListNode(-1);
           start.next=head;
           ListNode slow=start,fast=start;
           for(int i=1;i<=n;i++)
               fast=fast.next;
           while(fast.next!=null){
               fast=fast.next;
               slow=slow.next;
           }
           slow.next=slow.next.next;
           return start.next;
       }
   }
   ```

3. 给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。
   
   **你不能只是单纯的改变节点内部的值**，而是需要实际的进行节点交换。
   
   链接：https://leetcode-cn.com/problems/swap-nodes-in-pairs/
   
   思路：用三个指针，一个指向交换两节点的前方，其余指向要交换的两个，每次交换完三个指针前进两格
   
   ```java
   public ListNode swapPairs2(ListNode head) {
           ListNode start = new ListNode(-1);
           start.next = head;
           ListNode now=start, pre = start, prepre;
           while (pre.next != null && pre.next.next != null) {
               prepre = pre;
               pre = pre.next;
               now = pre.next;
               prepre.next = now;
               pre.next = now.next;
               now.next = pre;
           }
           return start.next;
       }
   ```

4. 给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置。
   
   链接：https://leetcode-cn.com/problems/rotate-list/
   
   思路：双指针，先让一个指针移到末尾，然后另一个指针移动k位，下一位断开，末尾指针接到头上
   
   ```java
   class Solution {
       public ListNode rotateRight(ListNode head, int k) {
           if (head==null||head.next==null) return head;
           ListNode dummy=new ListNode(-1);
           dummy.next=head;
           ListNode fast=dummy,slow=dummy;
           int length;
           for(length=0;fast.next!=null;length++)
               fast=fast.next;
           for(int i=length-k%length;i>0;i--)
               slow=slow.next;
           fast.next=dummy.next;
           dummy.next=slow.next;
           slow.next=null;
           return dummy.next;
       }
   }
   ```

5. 存在一个按升序排列的链表，给你这个链表的头节点 head ，请你删除链表中所有存在数字重复情况的节点，只保留原始链表中 没有重复出现 的数字。
   
   返回同样按升序排列的结果链表。
   
   来源：力扣（LeetCode）
   链接：https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii
   
   ```java
   class Solution {
       public ListNode rotateRight(ListNode head, int k) {
           if (head==null||head.next==null) return head;
           ListNode dummy=new ListNode(-1);
           dummy.next=head;
           ListNode fast=dummy,slow=dummy;
           int length;
           for(length=0;fast.next!=null;length++)
               fast=fast.next;
           for(int i=length-k%length;i>0;i--)
               slow=slow.next;
           fast.next=dummy.next;
           dummy.next=slow.next;
           slow.next=null;
           return dummy.next;
       }
   }
   ```

6. 给你一个链表的头节点 head 和一个特定值 x ，请你对链表进行分隔，使得所有 小于 x 的节点都出现在 大于或等于 x 的节点之前。
   
   你应当 保留 两个分区中每个节点的初始相对位置。
   链接：https://leetcode-cn.com/problems/partition-list
   
   思路：起两个新链表，拼起来
   
   ```java
   public ListNode partition(ListNode head, int x) {
           ListNode fakeNode1=new ListNode(-1);
           ListNode fakeNode2=new ListNode(-1);
           ListNode cur1=fakeNode1,cur2=fakeNode2;
           while(head!=null){
               if(head.val<x){
                   cur1.next=head;
                   cur1=head;
               }else{
                   cur2.next=head;
                   cur2=head;
               }
               head=head.next;
           }
           cur1.next=fakeNode2.next;
           cur2.next=null;
           return fakeNode1.next;
       }
   ```

7. 给你单链表的头指针 head 和两个整数 left 和 right ，其中 left <= right 。请你反转从位置 left 到位置 right 的链表节点，返回 反转后的链表 。
   
   来源：力扣（LeetCode）
   链接：https://leetcode-cn.com/problems/reverse-linked-list-ii
   
   ```java
   public ListNode reverseBetween(ListNode head, int m, int n) {
           ListNode fakeNode=new ListNode(-1);
           fakeNode.next=head;
           ListNode pre=new ListNode(-1);
           pre=fakeNode;
           for(int i=1;i<=m-1;i++)
               pre=pre.next;
           ListNode start=pre.next;
           ListNode then=start.next;
           for(int i=1;i<=n-m;i++){
               start.next=then.next;
               then.next=pre.next;
               pre.next=then;
               then=start.next;
           }
           return fakeNode.next;
       }
   ```

## 二叉树

1. 二叉树层次遍历
   
   之字形遍历时，只需要记一个标志然后翻转得到的数据或者使用双端队列头插或者尾插；
   
   从下往上添加时，只需要在结果列表插入时从头插入https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/

```java
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<TreeNode> q=new LinkedList<TreeNode>();
        List<List<Integer>> res=new LinkedList<>();
        if(root==null)return res;
        q.offer(root);        
        while(!q.isEmpty()){
            int qSize=q.size();    
            List<Integer> tempList=new LinkedList<>();
            for(int i=0;i<qSize;i++){               
                if(q.peek().left!=null)q.offer(q.peek().left);
                if(q.peek().right!=null)q.offer(q.peek().right);
                tempList.add(q.poll().val);
            }
            res.add(tempList);
        }
        return res;
    }
}
```

105.[从前序与中序遍历构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        return buildTreeHelper(0,0,inorder.length-1, preorder, inorder);
    }
    public TreeNode buildTreeHelper(int preStart,int inStart,int inEnd,int[] preorder, int[] inorder){
        if(preStart>preorder.length-1||inStart>inEnd)
            return null;
        TreeNode root=new TreeNode(preorder[preStart]);
        int inIndex=0;
        for(int i=0;i<inorder.length;i++){
            if(root.val==inorder[i])
                inIndex=i;
        }
        root.left=buildTreeHelper(preStart+1, inStart, inIndex-1, preorder, inorder);
        root.right=buildTreeHelper(preStart+inIndex-inStart+1, inIndex+1, inEnd, preorder, inorder);
        return root;
    }
}
```

中序与后续遍历恢复二叉树https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/submissions/

```java
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        return buildTreeHelper(postorder.length-1, 0, inorder.length-1, inorder, postorder);
    }
    public TreeNode buildTreeHelper(int postEnd,int inStart,int inEnd,int[] inorder, int[] postorder) {
        if(postEnd<0||inStart>inEnd)
            return null;
        TreeNode root=new TreeNode(postorder[postEnd]);
        int inIndex=0;
        for (int i = 0; i < inorder.length; i++)
            if(root.val==inorder[i])
                inIndex=i;
        root.left=buildTreeHelper(postEnd-(inEnd-inIndex)-1, inStart, inIndex-1, inorder, postorder);
        root.right=buildTreeHelper(postEnd-1, inIndex+1, inEnd, inorder, postorder);
        return root;
    }
}
```

[路径总和1](https://leetcode-cn.com/problems/path-sum)

给你二叉树的根节点 root 和一个表示目标和的整数 targetSum ，判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和 targetSum 。

```java
class Solution {
    public boolean hasPathSum(TreeNode root, int sum) {
        if(root==null)
                return false;
            sum-=root.val;
            if(root.left==null&&root.right==null&&sum==0)
                return true;
            return hasPathSum(root.left,sum)||hasPathSum(root.right,sum);
    }
}
```

[路径总和II](https://leetcode-cn.com/problems/path-sum-ii/)

求出每一条路径

```java
class Solution {
   public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>> result=new ArrayList<List<Integer>>();
        backtrack(root, sum, new ArrayList<>(), result);
        return result;
    }    
    public void backtrack(TreeNode start,int sum,List<Integer> tempList,List<List<Integer>> res){
        if(start==null)return;
        tempList.add(start.val);
        if(start.left==null&&start.right==null&&sum==start.val)
            res.add(new ArrayList<>(tempList));
        backtrack(start.left, sum-start.val, tempList, res);
        backtrack(start.right, sum-start.val, tempList, res);
        tempList.remove(tempList.size()-1);
    }
}
```

[路径总和III](https://leetcode-cn.com/problems/path-sum-iii)

给定一个二叉树的根节点 root ，和一个整数 targetSum ，求该二叉树里节点值之和等于 targetSum 的 路径 的数目。

```java
class Solution {
    public int pathSum(TreeNode root, int sum) {
        if(root==null)return 0;
        return dfs(root,sum)+pathSum(root.left,sum)+pathSum(root.right,sum);
    }
public int dfs(TreeNode root,int sum){
    if(root==null)return 0;
    int count=0;
    if(root.val==sum)
        count++;
    return count+dfs(root.left,sum-root.val)+dfs(root.right,sum-root.val);
}
}
```

[236. 二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

```java
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if(root==null||root==p||root==q)
            return root;
        TreeNode left=lowestCommonAncestor(root.left,p,q);
        TreeNode right=lowestCommonAncestor(root.right,p,q);
        if(left!=null&&right!=null)return root;
        if(left!=null)return left;
        if(right!=null)return right;
        return null;
    }
}
```

[199. 二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view/)

思路：本质是层次遍历的变种，放入队列时从右往左放，放入结果集时只放第一个就行

```java
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        Queue<TreeNode> queue=new LinkedList<TreeNode>();
        List<Integer> list=new ArrayList<Integer>();
        if(root==null)return list;
        queue.offer(root);
        while(!queue.isEmpty()){
            int size=queue.size();
            list.add(queue.peek().val);
            for(int i=0;i<size;i++){
                TreeNode temp=queue.poll();
                if(temp.right!=null)
                    queue.offer(temp.right);
                if(temp.left!=null)
                    queue.offer(temp.left);
            }
        }
        return list;
    }
}
```

[129. 求根节点到叶节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers/)

仍然用层次遍历，每次将父节点的数乘10然后赋给子节点，当节点的左右子树都为null时将其加入求和

```java
class Solution {
    public int sumNumbers(TreeNode root) {
        if(root==null)return 0;
         Queue<TreeNode> queue=new LinkedList<>();
        queue.offer(root);
        int sum=0;
        while(!queue.isEmpty()){
            int size=queue.size();           
            for(int i=0;i<size;i++){
                TreeNode temp=queue.poll();
                if(temp.left==null&&temp.right==null)
                    sum+=temp.val;
                else{
                    if(temp.left!=null){
                        temp.left.val=10*temp.val+temp.left.val;
                        queue.offer(temp.left);
                    }
                    if(temp.right!=null){
                        temp.right.val=10*temp.val+temp.right.val;
                        queue.offer(temp.right);
                    }
                }
            }
        }
        return sum;
    }
}
```

[98. 验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree/)

```java
class Solution {
   public boolean isValidBST(TreeNode root) {
        return isValidBST(root,Long.MIN_VALUE,Long.MAX_VALUE);
    }
    public boolean isValidBST(TreeNode root,long minVal,long maxVal){
        if(root==null)return true;
        if(root.val>=maxVal||root.val<=minVal)return false;
        return isValidBST(root.left, minVal, root.val)&&isValidBST(root.right,root.val,maxVal);
    }
}
```

[958. 二叉树的完全性检验](https://leetcode-cn.com/problems/check-completeness-of-a-binary-tree/)

```java
class Solution {
    public boolean isCompleteTree(TreeNode root) {
        LinkedList<TreeNode> q = new LinkedList<>();
        TreeNode cur;
        q.addLast(root);
        while ((cur = q.removeFirst()) != null) {
            q.addLast(cur.left);
            q.addLast(cur.right);
        }
        while (!q.isEmpty()) {
            if (q.removeLast() != null) {
                return false;
            }
        }
        return true;
    }
}
```

[662. 二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree/)

```java
public int widthOfBinaryTree(TreeNode root) {
        if (root == null) {
            return 0;
        }
        LinkedList<TreeNode> queue = new LinkedList();
        root.val = 1;
        queue.addLast(root);
        int maxWidth = 1;
        while (!queue.isEmpty()) {
            int size = queue.size();
            int beginIndex = -1;
            for (int i = 0; i < size; i++) {
                if (queue.peek() != null) {
                    TreeNode treeNode = queue.peek();
                    if (treeNode.left != null) {
                        treeNode.left.val = treeNode.val * 2;
                        queue.offer(treeNode.left);
                    }
                    if (treeNode.right != null) {
                        treeNode.right.val = treeNode.val * 2 + 1;
                        queue.offer(treeNode.right);
                    }
                }
                if (beginIndex == -1) {
                    beginIndex = queue.pop().val;
                } else {
                    maxWidth = Math.max(queue.pop().val - beginIndex + 1, maxWidth);
                }
            }
        }
        return maxWidth;
    }
```

[230. 二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst/)

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        if(count(root.left)==k-1)//如果左子树的节点刚好是K+1,那根节点就是要找的节点
            return root.val;
        else if(count(root.left)<k-1)//如果左子树的节点比K-1个要少，那就在右子树上找
            return kthSmallest(root.right,k-count(root.left)-1);
        else//否则在左子树上找
            return kthSmallest(root.left,k);
    }
    public int count(TreeNode root){//左右子树一共有多少个节点
        if(root==null)return 0;
        return 1+count(root.left)+count(root.right);
    }
}
```

[114. 二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list/)

```java
class Solution {
    public void flatten(TreeNode root) {
        while(root!=null){       
            if(root.left!=null){//左子树不为空，就把右子树接到左子树最后访问的节点上
                TreeNode cur=root.left;
                while(cur.right!=null)//找到最后访问的节点
                    cur=cur.right;
                cur.right=root.right;//把右子树接到左子树最后访问的节点上
                root.right=root.left;//把左子树接到又子树上
                root.left=null;//置空左子树
            }
            root=root.right;//往右子树继续重复上述步骤
        }
    }
}
```

二叉树的中序遍历

```java
public class InorderTraversal {
    //使用递归方法遍历
    public List<Integer> inorderTraversal1(TreeNode root) {
        List<Integer> res=new ArrayList<Integer>();
        inorderTraversalHelp(root,res);
        return res;
    }
    public void inorderTraversalHelp(TreeNode root,List<Integer> res) {
        if(root!=null){
            inorderTraversalHelp(root.left,res);
            res.add(root.val);
            inorderTraversalHelp(root.right, res);
        }
    }
    //使用非递归方法遍历
    public List<Integer> inorderTraversal2(TreeNode root) {
        Stack<TreeNode> s=new Stack<TreeNode>();
        List<Integer> l=new ArrayList<Integer>();
        TreeNode cur=root;
        while(!s.isEmpty()||root!=null){
            while(cur!=null){
                s.push(cur);
                cur=cur.left;
            }
            cur=s.pop();
            l.add(cur.val);
            cur=cur.right;
        }
        return l;
    }
}
```

## DFS

框架：

```
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return

    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

113. 二叉树路径总和

https://leetcode-cn.com/problems/path-sum-ii/

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
   public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>> result=new ArrayList<List<Integer>>();
        backtrack(root, sum, new ArrayList<>(), result);
        return result;
    }    
    public void backtrack(TreeNode start,int sum,List<Integer> tempList,List<List<Integer>> res){
        if(start==null)return;
        tempList.add(start.val);
        if(start.left==null&&start.right==null&&sum==start.val)
            res.add(new ArrayList<>(tempList));
        backtrack(start.left, sum-start.val, tempList, res);
        backtrack(start.right, sum-start.val, tempList, res);
        tempList.remove(tempList.size()-1);
    }
}
```

695. 岛屿最大面积
     
     https://leetcode-cn.com/problems/max-area-of-island/
     
     ```java
     class Solution {
         public int maxAreaOfIsland(int[][] grid) {
             int maxArea=0;
             for(int i=0;i<grid.length;i++)
                 for(int j=0;j<grid[0].length;j++)
                     if(grid[i][j]==1)maxArea=Math.max(maxArea, maxAreaOfIslandHelper(grid,i,j));
             return maxArea;
       }
     public int maxAreaOfIslandHelper(int[][] grid,int i,int j){
         if(i>=0&&i<grid.length&&j>=0&&j<grid[0].length&&grid[i][j]==1){
             grid[i][j]=0;
             return 1+maxAreaOfIslandHelper(grid,i+1,j)+maxAreaOfIslandHelper(grid,i-1,j)+maxAreaOfIslandHelper(grid,i,j+1)+maxAreaOfIslandHelper(grid,i,j-1);
                 }
         return 0;
     }
     }
     ```

## 回文字符串

- [[LeetCode\] 9. Palindrome Number 验证回文数字](https://www.cnblogs.com/grandyang/p/4125510.html)
  
  - 121是一个回文数字，-121不是，给定一个数字判断是否是回文数字，而且不允许转字符串
  
  - 思路：求出位数，然后通过斜杠求除法，%求余数，依次对比
    
    ```java
    public boolean isPalindrome(int x) {
            if(x<0) return false;
            int n=1;
            while(x/n>10){//求那么多位数的0
                n*=10;
            }
            while(x>0){
                int low=x%10;//低位
                int high=x/n;//高位
                if(low!=high)return false;
                x=(x%n)/10;//剥离前后位
                n=n/100;//剥离两个0
            }
            return true;
    }
    ```

- [[LeetCode\] Valid Palindrome 验证回文字符串](https://www.cnblogs.com/grandyang/p/4030114.html)
  
  - "A man, a plan, a canal: Panama"是回文字符串`"race a car"` 不是回文字符串
  
  - 给定一个字符串，判断是否为回文字符串
  
  - 使用双指针，遇到不是字符串的跳过即可-

## 动态规划

[5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

```java
public class testdama {
    public static void main(String[] args) {
        String str="babad";
        int strLength=str.length();
        if(strLength<2){
            System.out.println(str);
        }
        int maxLen=1;
        int begin=0;
        boolean[][] dp=new boolean[strLength][strLength];
        for(int i=strLength-1;i>=0;i--){
            for(int j=i;j<strLength;j++){
                if(i==j){
                    //单个字母一定是回文字符串
                    dp[i][j]=true;
                }else {
                    //字母i与字母j一样，长度小于2时一定是回文，长度不小于2那就追溯退掉这俩字母的中间串;核心逻辑
                    dp[i][j] = (str.charAt(i) == str.charAt(j)) && (j - i + 1 <= 2 || dp[i + 1][j - 1]);
                }
                if (dp[i][j] && j - i + 1 > maxLen) {
                    maxLen = j - i + 1;//标记最长
                    begin = i;//标记开始位置
                }
            }
        }
        for(int i=0;i<strLength;i++){
            for(int j=0;j<strLength;j++){
                System.out.print(dp[i][j]+" ");
            }
            System.out.println();
        }
        System.out.println(str.substring(begin,begin+maxLen));
    }
}
```

[221. 最大正方形](https://leetcode-cn.com/problems/maximal-square)

```java
public class testdama {
    public static void main(String[] args) {
        char[][] matrix=new char[][]{
                {'1','0','1','0','0'},
                {'1','0','1','1','1'},
                {'1','1','1','1','1'},
                {'1','0','0','1','0'}
        };
        if(matrix.length==0||matrix[0].length==0){
            System.out.println(-1);
        }
        int dp[][]=new int[matrix.length][matrix[0].length];
        int max=0;
        for(int i=0;i<matrix.length;i++){
            for(int j=0;j<matrix[0].length;j++){
                if(matrix[i][j]=='1') {
                    if (i == 0 || j == 0) {
                        dp[i][j]=1;
                    }else {
                        //取其上边，左边，左上边的最小值
                        dp[i][j]=Math.min(Math.min(dp[i][j-1],dp[i-1][j]),dp[i-1][j-1])+1;
                    }
                }
                max=Math.max(max, dp[i][j]*dp[i][j]);
            }
        }

        for(int i=0;i<matrix.length;i++){
            for(int j=0;j<matrix[0].length;j++){
                System.out.print(dp[i][j]+" ");
            }
            System.out.println();
        }
        System.out.println(max);
    }
}
```

[322. 零钱兑换](https://leetcode-cn.com/problems/coin-change)

```java
public class testdama {
    public static void main(String[] args) {
        int amount=27;
        int[] coins=new int[]{2,5,10,1};
        int[] dp=new int[amount+1];
        Arrays.fill(dp,amount+1);//用来判断能否满足条件
        dp[0]=0;
        for(int i=1;i<=amount;i++){
            for(int j=0;j<coins.length;j++){
                if(i>=coins[j]){
                    dp[i]=Math.min(dp[i], dp[i-coins[j]]+1);
                }
            }
        }
        for(int i=0;i<dp.length;i++){
            System.out.print(dp[i]+" ");
        }
        System.out.println(dp[amount]>amount?-1:dp[amount]);
    }
}
```

[圆环回原点问题](https://mp.weixin.qq.com/s/NZPaFsFrTybO3K3s7p7EVg)

- 0-12共13个数构成一个环，从0出发，每次走1步，走n步回到0共有多少种走法

青蛙跳台阶的变种问题

```java
/**
 * dp[i][j]表示从0走i步到j，要走4步到0，先算走3步到1或者走3步到9
 * */
public class testdama {
    public static void main(String[] args) {
        int circleLength = 10;
        int step = 4;
        int[][] dp=new int[step + 1][circleLength];
        dp[0][0] = 1;
        for(int i = 1; i <= step; i++){
            for(int j = 0; j < circleLength; j++){
                dp[i][j] = dp[i - 1][(j + 1) % circleLength] + dp[i - 1][(j - 1 + circleLength) % circleLength];
            }
        }
        for(int i = 0; i <= step; i++){
            for(int j = 0; j < circleLength; j++){
                System.out.print(dp[i][j]+" ");
            }
            System.out.println();
        }
        System.out.println(dp[step][0]);
    }
}
```

1143. [最长公共子序列](https://leetcode-cn.com/problems/longest-common-subsequence/)

```java
/**
dp[i][j]标识从text1的0~i-1，从text2的0~j-1的最长公共子序列的长度
**/
public static void main(String[] args) {
        String text1 = "bsbininm";
        String text2 = "jmjkbkjkv";
        int dp[][] = new int[text1.length() + 1][text2.length() + 1];
        for (int i = 1; i <= text1.length(); i++) {
            for (int j = 1; j <= text2.length(); j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]);
                }
            }
        }
        for (int i = 0; i <= text1.length(); i++) {
            for (int j = 0; j <= text2.length(); j++) {
                System.out.print(dp[i][j] + " ");
            }
            System.out.println();
        }
        System.out.println(dp[text1.length()][text2.length()]);
    }
```

[518. 零钱兑换 II](https://leetcode-cn.com/problems/coin-change-2/)

```java
 /**
 dp表示i块钱时，有的组成方法
 */
 public static void main(String[] args) {
         int amount=3;
         int[] coins=new int[]{2};
         int[] dp=new int[amount+1];
         dp[0]=1;
         for(int coin:coins){
             for(int i=coin;i<=amount;i++){
                 dp[i]+=dp[i-coin];
             }
         }
         System.out.println(dp[amount]);
     }
```

[718. 最长重复子数组](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)

这题实际上是求最长公共子串，注意与最长公共子序列的区别

```java
public static void main(String[] args) {
        int[] nums1 = new int[]{1, 2, 3, 2, 1};
        int[] nums2 = new int[]{3, 2, 1, 4, 7};
        int dp[][] = new int[nums1.length + 1][nums2.length + 1];
        int max=0;
        for (int i = 1; i <= nums1.length; i++) {
            for (int j = 1; j <= nums2.length; j++) {
                if (nums1[i - 1] == nums2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                    max=Math.max(max, dp[i][j]);
                } else {
                    dp[i][j] = 0;
                }
            }
        }
        for (int i = 1; i <= nums1.length; i++) {
            for (int j = 1; j <= nums2.length; j++) {
                System.out.print(dp[i][j] + " ");
            }
            System.out.println();
        }
        System.out.println(max);
    }
```

[139. 单词拆分](https://leetcode-cn.com/problems/word-break/)

```java
public static void main(String[] args) {
        String s = "applepenapple";
        List<String> wordDict = List.of("apple", "pen");
        Set<String> wordSet = new HashSet(wordDict);
        boolean[] dp = new boolean[s.length() + 1];
        dp[0] = true;
        for (int i = 1; i <= s.length(); i++) {
            for (int j = 0; j < i; j++) {
                //如果截取的子串是一个字典值并且在截取位置切割时前边的也是一个可切割值
                if (dp[j] && wordSet.contains(s.substring(j, i))) {
                    dp[i]=true;
                    break;
                }
            }
        }
        for (boolean d : dp) {
            System.out.print(d + " ");
        }
        //System.out.println(dp[s.length()]);
    }[673. 最长递增子序列的个数](https://leetcode-cn.com/problems/number-of-longest-increasing-subsequence/)
```

基本仿照最大数组和写，但是要额外保持一个尽可能小的负数积，因为负负得正

```java
public class testdama {
    public static void main(String[] args) {
        int[] nums=new int[]{2,3,-2,4};
        int[] dpMax=new int[nums.length];
        int[] dpMin=new int[nums.length];
        if(nums.length<=1)
            System.out.println(nums[0]);
        dpMax[0]=dpMin[0]=nums[0];
        for(int i=1;i<nums.length;i++){
            dpMax[i]=Math.max(dpMax[i - 1] * nums[i], Math.max(nums[i], dpMin[i - 1] * nums[i]));
            dpMin[i]=Math.min(dpMin[i - 1] * nums[i], Math.min(nums[i], dpMax[i - 1] * nums[i]));
        }

        for (int i = 0; i < nums.length; ++i) {
            System.out.print(dpMax[i]+" ");
        }
        System.out.println();
        for (int i = 0; i < nums.length; ++i) {
            System.out.print(dpMin[i]+" ");
        }
        System.out.println();

        int ans = dpMax[0];
        for (int i = 1; i < nums.length; ++i) {
            ans = Math.max(ans, dpMax[i]);
        }
        System.out.println(ans);
    }
}
```

[673. 最长递增子序列的个数](https://leetcode-cn.com/problems/number-of-longest-increasing-subsequence/)

```java
public class testdama {
    public static void main(String[] args) {
        int[] nums=new int[]{1,3,5,4,7};
        int[] dp=new int[nums.length];
        int[] cnt=new int[nums.length];
        for(int i=0;i<nums.length;i++){
            dp[i]=1;
            cnt[i]=1;
            for(int j=0;j<i;j++){
                if(nums[j]<nums[i]){
                    if(dp[j]+1>dp[i]){
                        //如果这里成立，表示加入第j个数成为了一个新的上升序列，此时应当刷新所有关于i的信息
                        dp[i]=dp[j]+1;
                        cnt[i]=cnt[j];
                    }else if(dp[j]+1==dp[i]){
                        //如果这里成立，表示到第i个数组成的上升序列加上第j个数是一个新的上升序列，比如之前有三个3，这有一个4，那所有的3的个数都满足条件
                        cnt[i]+=cnt[j];
                    }
                }
            }
        }
        int maxLen=0;
        int res=0;
        //找到那个最长的序列，然后把他们的达到数加起来
        for(int i=0;i<dp.length;i++){
            if(dp[i]>maxLen){
                maxLen=dp[i];
                res=cnt[i];
            }else if(dp[i]==maxLen){
                res+=dp[i];
            }
        }

        for(int i=0;i<dp.length;i++){
            System.out.print(dp[i]+" ");
        }
        System.out.println();
        for(int i=0;i<dp.length;i++){
            System.out.print(cnt[i]+" ");
        }
        System.out.println();

        System.out.println(res);
    }
}
```

## 双指针

