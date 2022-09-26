# 目录

>[仓库](https://github.com/github20131983/Algorithm)
>
>**链表**：
>
>1. [两数相加](https://leetcode-cn.com/problems/add-two-numbers)
>2. [删除链表倒数第n个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)
>3. [两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)
>
>**动态规划**
>
>1. 斐波那契数列
>   - [爬楼梯](https://leetcode.cn/problems/climbing-stairs/)
>   - [最小花费爬楼梯](https://leetcode.cn/problems/min-cost-climbing-stairs/)
>   - [变态爬楼梯](https://mp.weixin.qq.com/s/NZPaFsFrTybO3K3s7p7EVg)
>   - 强盗抢劫
>     - [打家劫舍](https://leetcode.cn/problems/house-robber/)
>     - [打家劫舍II(环形街区)](https://leetcode.cn/problems/house-robber-ii/)
>     - [打家劫舍III(二叉树街区)](https://leetcode.cn/problems/house-robber-iii/)
>   - 信件错排
>   - 母牛生产
>2. 矩阵
>   - 矩阵最小路径和
>   - 矩阵总路径数
>3. 数组区间
>   - [数组区间和](https://leetcode.cn/problems/range-sum-query-immutable/)
>   - [数组中等差递增子区间的个数](https://leetcode.cn/problems/arithmetic-slices/)
>4. 分割整数
>   - [分割整数的最大乘积](https://leetcode.cn/problems/integer-break/description/)
>   - [按平方数来分割整数](https://leetcode.cn/problems/perfect-squares/description/)
>   - [分割整数构成字母字符串](https://leetcode.cn/problems/decode-ways/description/)
>5. 子序列与子串
>   - [最长递增子序列的长度](https://leetcode.cn/problems/longest-increasing-subsequence/description/)
>   - [最长递增子序列的个数](https://leetcode-cn.com/problems/number-of-longest-increasing-subsequence/)
>   - [最长摆动子序列](https://leetcode.cn/problems/wiggle-subsequence/)
>   - [最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/submissions/)
>   - [最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)
>   - [最长数对列](https://leetcode.cn/problems/maximum-length-of-pair-chain/description/)
>   - [最长公共子串](https://leetcode-cn.com/problems/maximum-length-of-repeated-subarray/)
>   - [最大数组和](https://leetcode.cn/problems/maximum-subarray/)
>   - [最大数组积](https://leetcode.cn/problems/maximum-product-subarray/)
>   - [最大正方形](https://leetcode-cn.com/problems/maximal-square)
>6. 背包问题
>   - 简单背包
>   - [分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/description/)
>   - [目标和](https://leetcode.cn/problems/target-sum/description/)
>   - [01 字符构成最多的字符串](https://leetcode.cn/problems/ones-and-zeroes/description/)
>   - [找零钱的最少硬币数](https://leetcode.cn/problems/ones-and-zeroes/description/)
>   - [找零钱的硬币数组合](https://leetcode.cn/problems/coin-change-2/description/)
>   - [字符串按单词列表分割](https://leetcode.cn/problems/word-break/description/)
>   - [组合总和](https://leetcode.cn/problems/combination-sum-iv/description/)
>7. 买卖股票
>   - [买卖股票I](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)
>   - [买卖股票II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)
>   - [买卖股票III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)
>   - [买卖股票IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)
>   - [买卖股票V](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
>   - [买卖股票VI](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)
>8. 字符串编辑
>   - [删除两个字符串的字符使它们相等](https://leetcode.cn/problems/delete-operation-for-two-strings/description/)
>   - [编辑距离](https://leetcode.cn/problems/delete-operation-for-two-strings/description/)
>   - [复制粘贴字符](https://leetcode.cn/problems/2-keys-keyboard/description/)

# 链表

```java
public class ListNode {
   int val;
   ListNode next;
   ListNode(int x) { 
       val = x; 
   }
}
```

## 两数相加

> 给你两个非空的链表，表示两个非负的整数，数字逆序存储的，并且每个节点只能存储一位数字。请你将两个数相加，并以相同形式返回一个表示和的链表。
> **输入**：2->4->3+5->6->4  
> **输出**：7->0->8(**342 + 465 = 807**)
> **思路**：这个链表从前到后实际上是个位、十位、百位.......，所以从前往后刚好是模拟了从个位加到最高位的过程，基本上只要模拟就行了。
>
> ```java
> public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
>         ListNode sum=new ListNode(-1);
>         ListNode head=sum;
>         int sigleSum=0;
>         while(l1!=null||l2!=null){
>             sigleSum/=10;
>             if(l1!=null){
>                 sigleSum+=l1.val;
>                 l1=l1.next;
>             }
>             if(l2!=null){
>                 sigleSum+=l2.val;
>                 l2=l2.next;
>             }
>             sum.next=new ListNode(sigleSum%10);
>             sum=sum.next;
>         }
>         if(sigleSum>=10)sum.next=new ListNode(1);
>         return head.next;
>     }
> ```

## 删除链表倒数第n个节点

>给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点，使用一趟扫描实现
>**输入**：head = [1,2,3,4,5], n = 2
>**输出**：[1,2,3,5]
>**思路**：双指针，快的先跑n个，慢的在开始跑，当快的跑到末尾时，慢的指的就是要删除的
>
>```java
>class Solution {
>    public ListNode removeNthFromEnd(ListNode head, int n) {
>        ListNode start=new ListNode(-1);
>        start.next=head;
>        ListNode slow=start,fast=start;
>        for(int i=1;i<=n;i++)
>            fast=fast.next;
>        while(fast.next!=null){
>            fast=fast.next;
>            slow=slow.next;
>        }
>        slow.next=slow.next.next;//删除
>        return start.next;
>    }
>}
>```

## 两两交换链表中的节点

> 给定一个链表，**两两交换** **(换节点而不是值)** 其中相邻的节点，并返回交换后的链表。
> **思路**：用三个指针，一个指向交换两节点的前方，其余指向要交换的两个，每次交换完三个指针前进两格
>
> ```java
> public ListNode swapPairs2(ListNode head) {
>         ListNode start = new ListNode(-1);
>         start.next = head;
>         ListNode now=start, pre = start, prepre;
>         while (pre.next != null && pre.next.next != null) {
>             prepre = pre;
>             pre = pre.next;
>             now = pre.next;
>             prepre.next = now;
>             pre.next = now.next;
>             now.next = pre;
>         }
>         return start.next;
> }
> ```

1. 给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置。

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

2. 存在一个按升序排列的链表，给你这个链表的头节点 head ，请你删除链表中所有存在数字重复情况的节点，只保留原始链表中 没有重复出现 的数字。

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

3. 给你一个链表的头节点 head 和一个特定值 x ，请你对链表进行分隔，使得所有 小于 x 的节点都出现在 大于或等于 x 的节点之前。

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

4. 给你单链表的头指针 head 和两个整数 left 和 right ，其中 left <= right 。请你反转从位置 left 到位置 right 的链表节点，返回 反转后的链表 。

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

# 二叉树

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

# 深度优先&广度优先

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

# 动态规划

## 斐波那契数列

### 爬楼梯

>爬楼梯。需要 `n` 阶你才能到达楼顶。每次你可以爬 `1` 或 `2` 个台阶。你有多少种不同的方法可以爬到楼顶
>
>```java
>f[i]=f[i-1]+f[i-2];//本质斐波那契
>```

### 最小花费爬楼梯

>一个整数数组 cost ，其中 cost[i] 是从楼梯第 i 个台阶向上爬需要支付的费用。一旦你支付此费用，即可选择向上爬一个或者两个台阶。
>
>```java
>class Solution {
>    public int minCostClimbingStairs(int[] cost) {
>        if(cost.length==1)  return cost[0];
>        if(cost.length==2)	return Math.min(cost[0], cost[1]);
>        int[] dp=new int[cost.length];
>        dp[0]=cost[0];
>        dp[1]=cost[1];
>        for(int i=2;i<cost.length;i++)
>        	dp[i]=Math.min(dp[i-2]+cost[i], dp[i-1]+cost[i]);
>        return Math.min(dp[cost.length-1],dp[cost.length-2]);
>    }
>}
>```
>
>

### 变态爬楼梯

>- 0-12共13个数构成一个环，从0出发，每次走1步，走n步回到0共有多少种走法
>
>青蛙跳台阶的变种问题
>
>```java
>/**
> * dp[i][j]表示从0走i步到j，要走4步到0，先算走3步到1或者走3步到9
> * */
>public class testdama {
>    public static void main(String[] args) {
>        int circleLength = 10;
>        int step = 4;
>        int[][] dp=new int[step + 1][circleLength];
>        dp[0][0] = 1;
>        for(int i = 1; i <= step; i++){
>            for(int j = 0; j < circleLength; j++){
>                dp[i][j] = dp[i - 1][(j + 1) % circleLength] + dp[i - 1][(j - 1 + circleLength) % circleLength];
>            }
>        }
>        for(int i = 0; i <= step; i++){
>            for(int j = 0; j < circleLength; j++){
>                System.out.print(dp[i][j]+" ");
>            }
>            System.out.println();
>        }
>        System.out.println(dp[step][0]);
>    }
>}
>```
>
>

### 强盗抢劫

- 打家劫舍

>```java
>dp[i]=Math.max(dp[i-2]+nums[i],dp[i-1]);
>```

- 打家劫舍II

>环形打劫
>
>```java
>算[0...n-2]与[1...n-1]的打劫情况
>```
>
>

- 打家劫舍III

>二叉树打劫
>
>```java
>public int rob(TreeNode root) {
>    int[] result = robInternal(root);
>    return Math.max(result[0], result[1]);
>}
>
>public int[] robInternal(TreeNode root) {
>    if (root == null) return new int[2];
>    int[] result = new int[2];
>
>    int[] left = robInternal(root.left);
>    int[] right = robInternal(root.right);
>    //0代表不偷当前节点，1代表偷当前节点
>  	//不偷当前节点,左孩子能偷到的钱 + 右孩子能偷到的钱
>    result[0] = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);
>    //偷当前节点,左孩子选择自己不偷时能得到的钱 + 右孩子选择不偷时能得到的钱 + 当前节点的钱数
>    result[1] = left[0] + right[0] + root.val;
>
>    return result;
>}
>```
>
>

### 信件错排

>有 N 个 信 和 信封，它们被打乱，求错误装信方式的数量
>
>**思路**：要么前面i-1个都错了，那就随便换一下当前的，有i-1种换法，要么前面只有一封错了，那就只能跟那一封换也有i-1种换法，dp[i]代表i=封信全部错排的方法数
>
>```java
>dp[i]=(i-1)(dp[i-1]+dp[i-2])
>```
>
>

### 母牛生产

>母牛每年都会生 1 头小母牛，并且永远不会死。第一年有 1 只小母牛，从第二年开始，母牛开始生小母牛。每只小母牛 3 年之后成熟又可以生小母牛。给定整数 N，求 N 年后牛的数量。
>
>**思路**：这跟兔子生产一样，斐波那契数列最初就是研究兔子生产关系的
>
>```java
>dp[i]=dp[i-1]+dp[i-3]
>```
>
>

## 矩阵路径

### 矩阵最小路径和

>从矩阵左上角走到矩阵右下角的最短路径和
>
>`dp[i][j]=dp[i-1][j]+dp[i][j-1]`

### 矩阵总路径数

>统计从矩阵左上角到右下角的路径总数
>
>`dp[i][j]=dp[i-1][j]+dp[i][j-1]`

## 数组区间

### 数组区间和

>求区间 i ~ j 的和，可以转换为 sum[j + 1] - sum[i]

### 数组中等差递增子区间的个数

>```java
>dp[i] 表示以 A[i] 为结尾的等差递增子区间的个数。
>
>当 A[i] - A[i-1] == A[i-1] - A[i-2]，那么 [A[i-2], A[i-1], A[i]] 构成一个等差递增子区间。而且在以 A[i-1] 为结尾的递增子区间的后面再加上一个 A[i]，一样可以构成新的递增子区间。
>```
>
>```java
>public int numberOfArithmeticSlices(int[] A) {
>    if (A == null || A.length == 0) {
>        return 0;
>    }
>    int n = A.length;
>    int[] dp = new int[n];
>    for (int i = 2; i < n; i++) {
>        if (A[i] - A[i - 1] == A[i - 1] - A[i - 2]) {
>           //dp[i - 1]以i-1结尾能构成子区间的都能以i结尾构成子区间，
>           //加的1表示A[i-2], A[i-1], A[i]这个以前不是的现在也是了
>            dp[i] = dp[i - 1] + 1;
>        }
>    }
>    int total = 0;
>    for (int cnt : dp) {
>        total += cnt;
>    }
>    return total;
>}
>```
>
>

## 分割整数

### 整数拆分

>**输入**: n = 10
>**输出**: 36
>**解释**: 10 = 3 + 3 + 4, 3 × 3 × 4 = 36。
>
>```java
>class Solution {
>    public int integerBreak(int n) {
>        int[] dp=new int[n+1];
>        dp[1]=1;
>        for(int i=2;i<=n;i++)
>          for(int j=1;j<=i-1;j++){//注意至少要拆成2个数，那就是每个数都在1~n-1之间
>            //看看是之前形成的分割更大还是新形成的分割更大
>            //新形成的分割看看是j*(i-j), j*dp[i-j]哪个更大
>            //比如分割10的时候，刚刚算完了j=6为分割点的最大值，现在到7为分割点了，看是7*3大还是7*dp[3]大
>            dp[i]=Math.max(dp[i],Math.max(j*(i-j), j*dp[i-j]));
>          }
>        return dp[n];
>    }
>}
>```
>
>

### 按照平方数分割整数

>输入：n = 12
>输出：3 
>解释：12 = 4 + 4 + 4
>
>```java
>class Solution {
>    public int numSquares(int n) {
>        int[] dp = new int[n + 1];
>        for (int i = 1; i <= n; i++) {//对于1-n的每一个数都算出组成它的最小平方分割数
>            dp[i] = i;
>            for (int j = 1; j * j <= i; j++) {
>                //对于1到j*j小于i的j，切为一个值的平方与i - j * j的最小切割,加的1是可以用切为的那个平方
>                dp[i] = Math.min(dp[i], dp[i - j * j] + 1);
>            }
>        }
>        return dp[n];
>    }
>}
>//比如对于10来说，是切为1，dp[9]或者4，dp[6]或者9，dp[1]
>```
>
>

### 分割整数构成字符串（解码）

>输入：s = "226"
>输出：3
>解释：它可以解码为 "BZ" (2 26), "VF" (22 6), 或者 "BBF" (2 2 6)
>
>```java
>public int numDecodings(String s) {
>    if (s == null || s.length() == 0) {
>        return 0;
>    }
>    int n = s.length();
>    int[] dp = new int[n + 1];//表示截止到某一位的解码方法数
>    dp[0] = 1;
>    dp[1] = s.charAt(0) == '0' ? 0 : 1;//第一个如果是0的话不能解码
>    for (int i = 2; i <= n; i++) {
>        int one = Integer.valueOf(s.substring(i - 1, i));//切出当前位
>        if (one != 0) {
>            dp[i] += dp[i - 1];
>        }
>        if (s.charAt(i - 2) == '0') {
>            continue;
>        }
>        int two = Integer.valueOf(s.substring(i - 2, i));//切出当前位加上当前位的前一位
>        if (two <= 26) {
>            dp[i] += dp[i - 2];
>        }
>    }
>    return dp[n];
>}
>```
>
>

## 子序列与子串

>子序列不连续，子串连续

### 最长递增子序列的长度

>输入：nums = [10,9,2,5,3,7,101,18]
>输出：4
>解释：最长递增子序列是 [2,3,7,101]，因此长度为 4 。
>
>```java
>class Solution {
>    public int lengthOfLIS(int[] nums) {
>        if(nums.length<1)
>            return -1;
>        int[] dp=new int[nums.length];
>        int max=1;
>        for(int i=0;i<nums.length;i++){
>            for(int j=0;j<i;j++){
>                if(nums[j]<nums[i]){
>                    dp[i]=Math.max(dp[i],dp[j]+1);
>                }
>            }
>            if(dp[i]==0){
>                dp[i]=1;
>            }
>            max=Math.max(dp[i],max);
>        }
>        return max;
>    }
>}
>```
>
>

### 最长递增子序列的个数

>输入: [1,3,5,4,7]
>输出: 2
>解释: 有两个最长递增子序列，分别是 [1, 3, 4, 7] 和[1, 3, 5, 7]。
>
>```java
>public class Solution {
>    public static void main(String[] args) {
>        int[] nums=new int[]{1,3,5,4,7};
>        int[] dp=new int[nums.length];//到当前值的上升子序列的长度
>        int[] cnt=new int[nums.length];//到当前值的上升子序列的个数
>        for(int i=0;i<nums.length;i++){
>            dp[i]=1;
>            cnt[i]=1;
>            for(int j=0;j<i;j++){
>                if(nums[j]<nums[i]){
>                    if(dp[j]+1>dp[i]){
>                        //如果这里成立，表示加入第j个数成为了一个新的上升序列，此时应当刷新所有关于i的信息
>                        dp[i]=dp[j]+1;
>                        cnt[i]=cnt[j];
>                    }else if(dp[j]+1==dp[i]){
>                        //如果这里成立，表示到第i个数组成的上升序列加上第j个数是一个新的上升序列，比如之前有三个3，这有一个4，那所有的3的个数都满足条件
>                        cnt[i]+=cnt[j];
>                    }
>                }
>            }
>        }
>        int maxLen=0;
>        int res=0;
>        //找到那个最长的序列，然后把他们的达到数加起来
>        for(int i=0;i<dp.length;i++){
>            if(dp[i]>maxLen){
>                maxLen=dp[i];//找到最长的
>                res=cnt[i];//同时用这个最长值对应的计数值刷新这个结果
>            }else if(dp[i]==maxLen){
>                res+=cnt[i];//如果当前值等于最长值，就加上它的计数
>            }
>        }
>
>        for(int i=0;i<dp.length;i++){
>            System.out.print(dp[i]+" ");
>        }
>        System.out.println();
>        for(int i=0;i<dp.length;i++){
>            System.out.print(cnt[i]+" ");
>        }
>        System.out.println();
>
>        System.out.println(res);
>    }
>}
>```
>
>

### 最长摆动子序列

>**输入**: [1,17,5,10,13,15,10,5,16,8]
>**输出**: 7
>**解释**： [1,17,10,13,10,16,8].
>
>```java
>public int wiggleMaxLength(int[] nums) {
>    if (nums == null || nums.length == 0) {
>        return 0;
>    }
>    int up = 1, down = 1;
>    for (int i = 1; i < nums.length; i++) {
>        if (nums[i] > nums[i - 1]) {
>            up = down + 1;
>        } else if (nums[i] < nums[i - 1]) {
>            down = up + 1;
>        }
>    }
>    return Math.max(up, down);
>}
>```
>
>

### 最长公共子序列

>**输入**：text1 = "abcde", text2 = "ace" 
>**输出**：3  
>**解释**：最长公共子序列是 "ace" ，它的长度为 3 。
>
>```java
>/**
>dp[i][j]标识从text1的0~i-1，从text2的0~j-1的最长公共子序列的长度
>**/
>public static void main(String[] args) {
>        String text1 = "bsbininm";
>        String text2 = "jmjkbkjkv";
>        int dp[][] = new int[text1.length() + 1][text2.length() + 1];
>        for (int i = 1; i <= text1.length(); i++) {
>            for (int j = 1; j <= text2.length(); j++) {
>                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
>                    dp[i][j] = dp[i - 1][j - 1] + 1;
>                } else {
>                    dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]);
>                }
>            }
>        }
>        for (int i = 0; i <= text1.length(); i++) {
>            for (int j = 0; j <= text2.length(); j++) {
>                System.out.print(dp[i][j] + " ");
>            }
>            System.out.println();
>        }
>        System.out.println(dp[text1.length()][text2.length()]);
>    }
>```
>
>

### 最长回文子串

>给你一个字符串 `s`，找到 `s`中最长的回文子串
>**输入**：s = "babad"
>**输出**："bab"
>**思路**：`dp[i][j]`表示从i到j的子串是否是一个回文，计算时从下到上，从左到右而且只算上半部分三角，那是因为计算[1..4]是不是需要依赖于[2..3]是不是，即右上角的值依赖于左下角的值
>
>```java
>public class solution {
>    public static void main(String[] args) {
>        String str="babad";
>        int strLength=str.length();
>        if(strLength<2){
>            System.out.println(str);
>        }
>        int maxLen=1;
>        int begin=0;
>        boolean[][] dp=new boolean[strLength][strLength];
>        for(int i=strLength-1;i>=0;i--){
>            for(int j=i;j<strLength;j++){
>                if(i==j){
>                    //单个字母一定是回文字符串
>                    dp[i][j]=true;
>                }else {
> //字母i与字母j一样，长度小于2时一定是回文，长度不小于2那就追溯退掉这俩字母的中间串;核心逻辑
>                    dp[i][j] = (str.charAt(i) == str.charAt(j)) && (j - i + 1 <= 2 || dp[i + 1][j - 1]);
>                }
>                if (dp[i][j] && j - i + 1 > maxLen) {
>                    maxLen = j - i + 1;//标记最长
>                    begin = i;//标记开始位置
>                }
>            }
>        }
>        for(int i=0;i<strLength;i++){
>            for(int j=0;j<strLength;j++){
>                System.out.print(dp[i][j]+" ");
>            }
>            System.out.println();
>        }
>        System.out.println(str.substring(begin,begin+maxLen));
>    }
>}
>```
>
>| （i，j） | 0b   | 1a   | 2b   | 3a   | 4d   |
>| -------- | ---- | ---- | ---- | ---- | ---- |
>| 0b       | T    |      | T    |      |      |
>| 1a       |      | T    |      | T    |      |
>| 2b       |      |      | T    |      |      |
>| 3a       |      |      |      | T    |      |
>| 4d       |      |      |      |      | T    |

### 最长数对列

>**输入**：[[1,2], [2,3], [3,4]]
>**输出**：2
>**解释**：最长的数对链是 [1,2] -> [3,4]
>
>```java
>public int findLongestChain(int[][] pairs) {
>    if (pairs == null || pairs.length == 0) {
>        return 0;
>    }
>    Arrays.sort(pairs, (a, b) -> (a[0] - b[0]));
>    int n = pairs.length;
>    int[] dp = new int[n];
>    Arrays.fill(dp, 1);
>    for (int i = 1; i < n; i++) {
>        for (int j = 0; j < i; j++) {
>            if (pairs[j][1] < pairs[i][0]) {
>                dp[i] = Math.max(dp[i], dp[j] + 1);
>            }
>        }
>    }
>    return Arrays.stream(dp).max().orElse(0);
>}
>```
>
>

### 最长公共子串

>给两个整数数组 nums1 和 nums2 ，返回 两个数组中 公共的 、长度最长的子数组的长度 。
>
>输入：nums1 = [1,2,3,2,1], nums2 = [3,2,1,4,7]
>输出：3
>解释：长度最长的公共子数组是 [3,2,1] 。
>
>```java
>public static void main(String[] args) {
>        int[] nums1 = new int[]{1, 2, 3, 2, 1};
>        int[] nums2 = new int[]{3, 2, 1, 4, 7};
>        int dp[][] = new int[nums1.length + 1][nums2.length + 1];
>        int max=0;
>        for (int i = 1; i <= nums1.length; i++) {
>            for (int j = 1; j <= nums2.length; j++) {
>                if (nums1[i - 1] == nums2[j - 1]) {
>                    dp[i][j] = dp[i - 1][j - 1] + 1;
>                    max=Math.max(max, dp[i][j]);
>                } else {
>                    dp[i][j] = 0;
>                }
>            }
>        }
>        for (int i = 1; i <= nums1.length; i++) {
>            for (int j = 1; j <= nums2.length; j++) {
>                System.out.print(dp[i][j] + " ");
>            }
>            System.out.println();
>        }
>        System.out.println(max);
>    }
>```
>
>

### 最大数组和

>给你一个整数数组 nums ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。子数组 是数组中的一个连续部分。
>
>输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
>输出：6
>解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
>
>```java
>class Solution {
>    public int maxSubArray(int[] nums) {
>        if (nums.length < 1)
>            return -1;
>        int max = nums[0];
>        int res = max;
>        for (int i = 1; i < nums.length; i++) {
>            if (max < 0) {
>                max = nums[i];
>            }else {
>                max+=nums[i];
>            }
>            res=Math.max(res, max);
>        }
>        return res;
>    }
>}
>```
>
>

### 最大数组积

>给你一个整数数组 nums ，请你找出数组中乘积最大的非空连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积。测试用例的答案是一个 32-位 整数。子数组 是数组的连续子序列。
>
>输入: nums = [2,3,-2,4]
>输出: 6
>解释: 子数组 [2,3] 有最大乘积 6。
>
>```java
>public class testdama {
>    public static void main(String[] args) {
>        int[] nums=new int[]{2,3,-2,4};
>        int[] dpMax=new int[nums.length];
>        int[] dpMin=new int[nums.length];
>        if(nums.length<=1)
>            System.out.println(nums[0]);
>        dpMax[0]=dpMin[0]=nums[0];
>        for(int i=1;i<nums.length;i++){
>            dpMax[i]=Math.max(dpMax[i - 1] * nums[i], Math.max(nums[i], dpMin[i - 1] * nums[i]));
>            dpMin[i]=Math.min(dpMin[i - 1] * nums[i], Math.min(nums[i], dpMax[i - 1] * nums[i]));
>        }
>
>        for (int i = 0; i < nums.length; ++i) {
>            System.out.print(dpMax[i]+" ");
>        }
>        System.out.println();
>        for (int i = 0; i < nums.length; ++i) {
>            System.out.print(dpMin[i]+" ");
>        }
>        System.out.println();
>
>        int ans = dpMax[0];
>        for (int i = 1; i < nums.length; ++i) {
>            ans = Math.max(ans, dpMax[i]);
>        }
>        System.out.println(ans);
>    }
>}
>```
>
>

### 最大正方形

>在一个由 `'0'` 和 `'1'` 组成的二维矩阵内，找到只包含 `'1'` 的最大正方形，并返回其面积。
>
>输入：matrix = [
>		["1","0","1","0","0"],
>		["1","0","1","1","1"],
>		["1","1","1","1","1"],
>		["1","0","0","1","0"]
>]
>输出：4
>
>```java
>public class testdama {
>    public static void main(String[] args) {
>        char[][] matrix=new char[][]{
>                {'1','0','1','0','0'},
>                {'1','0','1','1','1'},
>                {'1','1','1','1','1'},
>                {'1','0','0','1','0'}
>        };
>        if(matrix.length==0||matrix[0].length==0){
>            System.out.println(-1);
>        }
>        int dp[][]=new int[matrix.length][matrix[0].length];
>        int max=0;
>        for(int i=0;i<matrix.length;i++){
>            for(int j=0;j<matrix[0].length;j++){
>                if(matrix[i][j]=='1') {
>                    if (i == 0 || j == 0) {
>                        dp[i][j]=1;
>                    }else {
>                        //取其上边，左边，左上边的最小值
>                        dp[i][j]=Math.min(Math.min(dp[i][j-1],dp[i-1][j]),dp[i-1][j-1])+1;
>                    }
>                }
>                max=Math.max(max, dp[i][j]*dp[i][j]);
>            }
>        }
>
>        for(int i=0;i<matrix.length;i++){
>            for(int j=0;j<matrix[0].length;j++){
>                System.out.print(dp[i][j]+" ");
>            }
>            System.out.println();
>        }
>        System.out.println(max);
>    }
>}
>```
>
>

## 背包问题

### 简单背包

>有一个容量为 N 的背包，要用这个背包装下物品的价值最大，这些物品有两个属性：体积 w 和价值 v。
>
>定义一个二维数组 dp 存储最大价值，其中 dp[i][j] 表示前 i 件物品体积不超过 j 的情况下能达到的最大价值。设第 i 件物品体积为 w，价值为 v，根据第 i 件物品是否添加到背包中，可以分两种情况讨论：
>
>- 第 i 件物品没添加到背包，总体积不超过 j 的前 i 件物品的最大价值就是总体积不超过 j 的前 i-1 件物品的最大价值，dp[i][j] = dp[i-1][j]。
>- 第 i 件物品添加到背包中，`dp[i][j] = dp[i-1][j-w] + v。`
>
>```java
>// W 为背包总体积
>// N 为物品数量
>// weights 数组存储 N 个物品的重量
>// values 数组存储 N 个物品的价值
>public int knapsack(int W, int N, int[] weights, int[] values) {
>    int[][] dp = new int[N + 1][W + 1];
>    for (int i = 1; i <= N; i++) {
>        int w = weights[i - 1], v = values[i - 1];
>        for (int j = 1; j <= W; j++) {
>            if (j >= w) {
>                dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - w] + v);
>            } else {
>                dp[i][j] = dp[i - 1][j];
>            }
>        }
>    }
>    return dp[N][W];
>}
>```
>
>优化空间
>
>因为 `dp[j-w] `表示 `dp[i-1][j-w]`，因此不能先求` dp[i][j-w]`，防止将 `dp[i-1][j-w]` 覆盖。也就是说要先计算 `dp[i][j]` 再计算 `dp[i][j-w]`，在程序实现时需要按倒序来循环求解。
>
>```java
>public int knapsack(int W, int N, int[] weights, int[] values) {
>    int[] dp = new int[W + 1];
>    for (int i = 1; i <= N; i++) {
>        int w = weights[i - 1], v = values[i - 1];
>        for (int j = W; j >= 1; j--) {
>            if (j >= w) {
>                dp[j] = Math.max(dp[j], dp[j - w] + v);
>            }
>        }
>    }
>    return dp[W];
>}
>```
>
>

### 分割等和子集

>**输入**：nums = [1,5,11,5]
>**输出**：true
>**解释**：数组可以分割成 [1, 5, 5] 和 [11] 。
>
>```java
>//可以看成一个背包大小为 sum/2 的 0-1 背包问题。
>public boolean canPartition(int[] nums) {
>    int sum = computeArraySum(nums);
>    if (sum % 2 != 0) {
>        return false;
>    }
>    int W = sum / 2;
>    boolean[] dp = new boolean[W + 1];
>    dp[0] = true;
>    for (int num : nums) {                 // 0-1 背包一个物品只能用一次
>        for (int i = W; i >= num; i--) {   // 从后往前，先计算 dp[i] 再计算 dp[i-num]
>            dp[i] = dp[i] || dp[i - num];
>        }
>    }
>    return dp[W];
>}
>
>private int computeArraySum(int[] nums) {
>    int sum = 0;
>    for (int num : nums) {
>        sum += num;
>    }
>    return sum;
>}
>```
>
>

### 目标和

>**输入**：nums = [1,1,1,1,1], target = 3
>**输出**：5
>**解释**：一共有 5 种方法让最终目标和为 3 。
>-1 + 1 + 1 + 1 + 1 = 3
>+1 - 1 + 1 + 1 + 1 = 3
>+1 + 1 - 1 + 1 + 1 = 3
>+1 + 1 + 1 - 1 + 1 = 3
>+1 + 1 + 1 + 1 - 1 = 3
>
>```java
>//该问题可以转换为 Subset Sum 问题，从而使用 0-1 背包的方法来求解
>//只要找到一个子集，令它们都取正号，并且和等于 (target + sum(nums))/2，就证明存在解
>public int findTargetSumWays(int[] nums, int S) {
>    int sum = computeArraySum(nums);
>    if (sum < S || (sum + S) % 2 == 1) {
>        return 0;
>    }
>    int W = (sum + S) / 2;
>    int[] dp = new int[W + 1];
>    dp[0] = 1;
>    for (int num : nums) {
>        for (int i = W; i >= num; i--) {
>            dp[i] = dp[i] + dp[i - num];
>        }
>    }
>    return dp[W];
>}
>
>private int computeArraySum(int[] nums) {
>    int sum = 0;
>    for (int num : nums) {
>        sum += num;
>    }
>    return sum;
>}
>```
>
>

### 01 字符构成最多的字符串

>输入：strs = ["10", "0001", "111001", "1", "0"], m = 5, n = 3
>输出：4
>解释：最多有 5 个 0 和 3 个 1 的最大子集是 {"10","0001","1","0"} ，因此答案是 4 。
>其他满足题意但较小的子集包括 {"0001","1"} 和 {"10","1","0"} 。{"111001"} 不满足题意，因为它含 4 个 1 ，大于 n 的值 3 。
>
>```java
>//这是一个多维费用的 0-1 背包问题，有两个背包大小，0 的数量和 1 的数量。
>public int findMaxForm(String[] strs, int m, int n) {
>    if (strs == null || strs.length == 0) {
>        return 0;
>    }
>    int[][] dp = new int[m + 1][n + 1];
>    for (String s : strs) {    // 每个字符串只能用一次
>        int ones = 0, zeros = 0;
>        for (char c : s.toCharArray()) {
>            if (c == '0') {
>                zeros++;
>            } else {
>                ones++;
>            }
>        }
>        for (int i = m; i >= zeros; i--) {
>            for (int j = n; j >= ones; j--) {
>                dp[i][j] = Math.max(dp[i][j], dp[i - zeros][j - ones] + 1);
>            }
>        }
>    }
>    return dp[m][n];
>}
>```
>
>

### 找零钱的最少硬币数

>**输入**：coins = [1, 2, 5], amount = 11
>**输出**：3 
>**解释**：11 = 5 + 5 + 1
>
>```java
>public class Solution {
>    public static void main(String[] args) {
>        int amount=27;
>        int[] coins=new int[]{2,5,10,1};
>        int[] dp=new int[amount+1];
>        Arrays.fill(dp,amount+1);//用来判断能否满足条件
>        dp[0]=0;
>        for(int i=1;i<=amount;i++){
>            for(int j=0;j<coins.length;j++){
>                if(i>=coins[j]){
>                    dp[i]=Math.min(dp[i], dp[i-coins[j]]+1);
>                }
>            }
>        }
>        for(int i=0;i<dp.length;i++){
>            System.out.print(dp[i]+" ");
>        }
>        System.out.println(dp[amount]>amount?-1:dp[amount]);
>    }
>}
>```
>
>

### 找零钱的硬币数组合

>输入：amount = 5, coins = [1, 2, 5]
>输出：4
>解释：有四种方式可以凑成总金额：
>5=5
>5=2+2+1
>5=2+1+1+1
>5=1+1+1+1+1
>
>```java
>/**
> dp表示i块钱时，有的组成方法
> */
> public static void main(String[] args) {
>         int amount=3;
>         int[] coins=new int[]{2};
>         int[] dp=new int[amount+1];
>         dp[0]=1;
>         for(int coin:coins){
>             for(int i=coin;i<=amount;i++){
>                 dp[i]+=dp[i-coin];
>             }
>         }
>         System.out.println(dp[amount]);
>     }
>```
>
>

### 字符串按单词列表分割

>**输入**: s = "leetcode", wordDict = ["leet", "code"]
>**输出**: true
>**解释**: 返回 true 因为 "leetcode" 可以由 "leet" 和 "code" 拼接成。
>
>dict 中的单词没有使用次数的限制，因此这是一个完全背包问题。该问题涉及到字典中单词的使用顺序，也就是说物品必须按一定顺序放入背包中，例如下面的 dict 就不够组成字符串 "leetcode"：
>
>```html
>["lee", "tc", "cod"]
>```
>
>求解顺序的完全背包问题时，对物品的迭代应该放在最里层，对背包的迭代放在外层，只有这样才能让物品按一定顺序放入背包中。
>
>```java
>public boolean wordBreak(String s, List<String> wordDict) {
>    int n = s.length();
>    boolean[] dp = new boolean[n + 1];
>    dp[0] = true;
>    for (int i = 1; i <= n; i++) {
>        for (String word : wordDict) {   // 对物品的迭代应该放在最里层
>            int len = word.length();
>            if (len <= i && word.equals(s.substring(i - len, i))) {
>                dp[i] = dp[i] || dp[i - len];
>            }
>        }
>    }
>    return dp[n];
>}
>```
>
>

### 组合总和

>输入：nums = [1,2,3], target = 4
>输出：7
>解释：
>所有可能的组合为：
>(1, 1, 1, 1)
>(1, 1, 2)
>(1, 2, 1)
>(1, 3)
>(2, 1, 1)
>(2, 2)
>(3, 1)
>请注意，顺序不同的序列被视作不同的组合。
>
>涉及顺序的完全背包。
>
>```java
>public int combinationSum4(int[] nums, int target) {
>    if (nums == null || nums.length == 0) {
>        return 0;
>    }
>    int[] maximum = new int[target + 1];
>    maximum[0] = 1;
>    Arrays.sort(nums);
>    for (int i = 1; i <= target; i++) {
>        for (int j = 0; j < nums.length && nums[j] <= i; j++) {
>            maximum[i] += maximum[i - nums[j]];
>        }
>    }
>    return maximum[target];
>}
>```
>
>

## 买卖股票

[动态规划股票问题通解](https://xiaochen1024.com/courseware/60b4f11ab1aa91002eb53b18/61963bcdc1553b002e57bf13)

>**`dp[i][j][0,1]`代表第i天交易j次手中持有和不持有股票时能获得的最大利润**
>
>1. `dp[i][k][0]`  **第i天 交易k次 0手中没有股票**
>2. `dp[i][k][1]`  **第i天 交易k次 1手中有股票**
>
>
>
>**最大利润是以下两种情况的最大值**
>
>**今天没有持有股票，分为两种情况**：
>
>1. `dp[i - 1][k][0]` **昨天没有持有，今天不操作**
>2. `dp[i - 1][k][1] + prices[i]` **昨天持有，今天卖出，今天手中就没有股票了**
>
>**今天不持有股票的最大利润**: `dp[i][k][0] = Math.max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i])`
>
>
>
>**今天持有股票，分为两种情况**：
>
>1. `dp[i - 1][k][1] `**昨天持有，今天不操作**
>2. `dp[i - 1][k - 1][0] - prices[i] `**昨天没有持有，今天买入，注意在买入的时候记一次数**
>
>**今天持有股票的最大利润**：`dp[i][k][1] = Math.max(dp[i - 1][k][1], dp[i - 1][k - 1][0]-prices[i])`

### 买卖股票I

>只允许买卖一次，相当于k=1
>**思路**：
>
>>` dp[i][1][0] = Math.max(dp[i - 1][1][0], dp[i - 1][1][1] + prices[i])`
>>` dp[i][1][1] = Math.max(dp[i - 1][1][1], dp[i - 1][0][0] - prices[i])`
>>`Math.max(dp[i - 1][1][1], -prices[i])` // k=0时 没有交易次数，`dp[i - 1][0][0] `= 0
>>k是固定值1，不影响结果，所以可以不用管，简化之后如下：
>>`dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i])`
>>`dp[i][1] = Math.max(dp[i - 1][1], -prices[i])`
>>dp[i] 只和 dp[i - 1] 有关，可以去掉一维
>
>```java
>//交易次数1
>public class BuyAndSell121 {
>    //k固定是1，删去一维，i天只与i-1天有关，删去一维
>    public int maxProfit(int[] prices) {
>        int[] dp = new int[2];
>        dp[0] = 0;//第一天不持有时持有的利润
>        dp[1] = -prices[0];//第一天持有时持有的利润
>        for (int i = 1; i < prices.length; i++) {
>            //当前不持有，要么是前一天不持有，要么是前一天持有今天卖了
>            dp[0] = Math.max(dp[0], dp[1] + prices[i]);
>            //当前持有，要么是前一天持有，要么是前一天不持有今天买了，这里之所以用-prices[i]是因为k=0时没有交易次数
>            dp[1] = Math.max(dp[1], -prices[i]);
>        }
>        return dp[0];
>    }
>}
>```
>
>

### 买卖股票II

>交易次数无限制，相当于k = infinity
>
>**思路**：
>
>>` dp[i][k][0] = Math.max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i])`
>>` dp[i][k][1] = Math.max(dp[i - 1][k][1], dp[i - 1][k - 1][0] - prices[i])`
>> k不影响结果，简化之后如下:
>>` dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i])`
>>`dp[i][1] = Math.max(dp[i - 1][1], dp[i - 1][0] - prices[i])`
>>dp[i] 只和 dp[i - 1] 有关，可以去掉一维
>
>```java
>//交易次数无限制
>public class BuyAndSell122 {
>    public int maxProfit(int[] prices) {
>        int[] dp = new int[2];
>        dp[0] = 0;//不持有
>        dp[1] = -prices[0];//持有
>        for (int i = 1; i < prices.length; i++) {
>            //当前不持有，要么是前一天不持有，要么是前一天持有今天卖了
>            dp[0] = Math.max(dp[0], dp[1] + prices[i]);
>            ////当前持有，要么是前一天持有，要么是前一天不持有今天买了
>            dp[1] = Math.max(dp[1], dp[0] - prices[i]);
>        }
>        return dp[0];
>    }
>}
>```
>
>

### 买卖股票III

>只允许交易2次，k=2
>
>**思路**：
>
>>`dp[i][k][0] = Math.max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i])`
>>` dp[i][k][1] = Math.max(dp[i - 1][k][1], dp[i - 1][k - 1][0] - prices[i])`
>> k对结果有影响 不能舍去，只能对k进行循环
>>
>> ```java
>> for (let i = 0; i < n; i++) {
>>         for (let k = maxK; k >= 1; k--) {
>>        dp[i][k][0] = Math.max(dp[i - 1][k][0], dp[i - 1][k][1] + prices[i]);
>>        dp[i][k][1] = Math.max(dp[i - 1][k][1], dp[i - 1][k - 1][0] - prices[i]);
>>        }
>> }
>> ```
>>
>>k=2，直接写出循环的结果
>>
>>```java
>>dp[i][2][0] = Math.max(dp[i - 1][2][0], dp[i - 1][2][1] + prices[i])
>>dp[i][2][1] = Math.max(dp[i - 1][2][1], dp[i - 1][1][0] - prices[i])
>>
>>dp[i][1][0] = Math.max(dp[i - 1][1][0], dp[i - 1][1][1] + prices[i])
>>dp[i][1][1] = Math.max(dp[i - 1][1][1], dp[i - 1][0][0] - prices[i])
>>Math.max(dp[i - 1][1][1], -prices[i]) //k=0时 没有交易次数，dp[i - 1][0][0] = 0
>>```
>>
>>去掉i这一维度
>>
>>```java
>>dp[2][0] = Math.max(dp[2][0], dp[2][1] + prices[i])
>>dp[2][1] = Math.max(dp[2][1], dp[1][0] - prices[i])
>>
>>dp[1][0] = Math.max(dp[1][0], dp[1][1] + prices[i])
>>dp[1][1] = Math.max(dp[1][1], dp[0][0] - prices[i])
>>Math.max(dp[1][1], -prices[i]) //k=0时 没有交易次数，dp[i - 1][0][0] = 0
>>```
>
>```java
>//k=2
>public class BuyAndSell123 {
>    //[3,3,5,0,0,3,1,4]
>    public int maxProfit(int[] prices) {
>        int[][] dp = new int[3][2];
>        dp[2][1]=dp[1][1]=-prices[0];
>        dp[2][0]=dp[1][0]=0;
>        for (int i = 1; i < prices.length; i++) {
>            dp[2][0] = Math.max(dp[2][0], dp[2][1] + prices[i]);
>            dp[2][1] = Math.max(dp[2][1], dp[1][0] - prices[i]);
>
>            dp[1][0] = Math.max(dp[1][0], dp[1][1] + prices[i]);
>            // k=0时 没有交易次数，dp[i - 1][0][0] = 0
>            dp[1][1] = Math.max(dp[1][1], -prices[i]);
>        }
>        return dp[2][0];
>    }
>}
>```
>
>

### 买卖股票IV

>限定交易次数 最多次数为 k
>
>**思路**：
>
>>采用完整的递推策略
>
>```java
>//可以交易k次
>public class BuyAndSell188 {
>    public int maxProfit(int[] prices, int k) {
>        //[第几天][还可以交易k次][不持有0，持有1]，dp代表当前能获得的最大利润
>        int[][][] dp = new int[prices.length][k + 1][2];
>        for (int j = 0; j <= k; j++) {
>            dp[0][j][0] = 0;//第一天不持有
>            dp[0][j][1] = -prices[0];//第一天持有
>        }
>        for (int i = 1; i < prices.length; i++) {
>            for (int j = 1; j <= k; j++) {
>                //不持有股票要么是前一天就不持有，要么是前一天持有今天卖掉
>                dp[i][j][0] = Math.max(dp[i - 1][j][0], dp[i - 1][j][1] + prices[i]);
>                //持有股票要么是前一天就不持有，要么是前一天不持有今天买了，买的时候要把次数扣减
>                dp[i][j][1] = Math.max(dp[i - 1][j][1], dp[i - 1][j - 1][0] - prices[i]);
>            }
>        }
>        int ans = 0;
>        for (int j = 0; j <= k; j++) {
>            ans = Math.max(ans, dp[prices.length - 1][j][0]);
>        }
>        return ans;
>    }
>}
>```
>
>

### 买卖股票V

> 含有冷冻期，无限次交易
>
> **思路**：
>
> >每次买入时从前天开始
>
> ```java
> public class BuyAndSell309 {
>     public int maxProfit(int[] prices) {
>         int[][] dp = new int[prices.length][2];
>         dp[0][0] = 0;//第一天不持有
>         dp[0][1] = -prices[0];//第一天持有
> 
>         dp[1][0] = Math.max(dp[0][0], prices[1] + dp[0][1]);//第二天不持有
>         dp[1][1] = Math.max(-prices[1], dp[0][1]);//第二天持有
> 
>         for (int i = 2; i < prices.length; i++) {
>             // 当前不持有，要么是前一天不持有，要么是前一天持有今天卖了
>             dp[i][0] = Math.max(dp[i - 1][0], prices[i] + dp[i - 1][1]);
>             //今天持有，要么是昨天不持有，要么是前天不持有今天买了，注意在此
>             dp[i][1] = Math.max(dp[i - 2][0] - prices[i], dp[i - 1][1]);
>         }
>         return dp[prices.length - 1][0];
>     }
> }
> ```
>
> 

### 买卖股票VI

>含有手续费，无限次交易
>
>**思路**：
>
>>每次买入时扣掉手续费
>>`dp[i][0] = max(dp[i - 1][0], dp[i - 1][1] + prices[i] - fee)`
>>`dp[i][1] = max(dp[i - 1][1], dp[i - 1][0] - prices[i])`
>>
>>与k无关，i只与i-1有关
>
>```java
>public class BuyAndSell714 {
>    public int maxProfit(int[] prices, int fee) {
>        int[] dp = new int[2];
>        dp[0] = 0;//不持有
>        dp[1] = -prices[0];//持有
>        for (int i = 1; i < prices.length; i++) {
>            //当前不持有，要么是前一天不持有，要么是前一天持有今天卖了
>            dp[0] = Math.max(dp[0], dp[1] + prices[i] - fee);
>            ////当前持有，要么是前一天持有，要么是前一天不持有今天买了
>            dp[1] = Math.max(dp[1], dp[0] - prices[i]);
>        }
>        return dp[0];
>    }
>}
>```
>
>

## 字符串编辑

### 删除两个字符串的字符使它们相等

>输入: word1 = "sea", word2 = "eat"
>输出: 2
>解释: 第一步将 "sea" 变为 "ea" ，第二步将 "eat "变为 "ea"
>
>```java
>//可以转换为求两个字符串的最长公共子序列问题。
>public int minDistance(String word1, String word2) {
>    int m = word1.length(), n = word2.length();
>    int[][] dp = new int[m + 1][n + 1];
>    for (int i = 1; i <= m; i++) {
>        for (int j = 1; j <= n; j++) {
>            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
>                dp[i][j] = dp[i - 1][j - 1] + 1;
>            } else {
>                dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]);
>            }
>        }
>    }
>    return m + n - 2 * dp[m][n];
>}
>```
>
>

### 编辑距离

>给你两个单词 word1 和 word2， 请返回将 word1 转换成 word2 所使用的最少操作数  。
>
>你可以对一个单词进行如下三种操作：
>
>插入一个字符
>删除一个字符
>替换一个字符
>
>输入：word1 = "intention", word2 = "execution"
>输出：5
>解释：
>intention -> inention (删除 't')
>inention -> enention (将 'i' 替换为 'e')
>enention -> exention (将 'n' 替换为 'x')
>exention -> exection (将 'n' 替换为 'c')
>exection -> execution (插入 'u')
>
>```java
>public int minDistance(String word1, String word2) {
>    if (word1 == null || word2 == null) {
>        return 0;
>    }
>    int m = word1.length(), n = word2.length();
>    int[][] dp = new int[m + 1][n + 1];
>    for (int i = 1; i <= m; i++) {
>        dp[i][0] = i;
>    }
>    for (int i = 1; i <= n; i++) {
>        dp[0][i] = i;
>    }
>    for (int i = 1; i <= m; i++) {
>        for (int j = 1; j <= n; j++) {
>            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
>                dp[i][j] = dp[i - 1][j - 1];
>            } else {
>                dp[i][j] = Math.min(dp[i - 1][j - 1], Math.min(dp[i][j - 1], dp[i - 1][j])) + 1;
>            }
>        }
>    }
>    return dp[m][n];
>}
>```
>
>

### 复制粘贴字符

>输入：3
>输出：3
>解释：
>最初, 只有一个字符 'A'。
>第 1 步, 使用 Copy All 操作。
>第 2 步, 使用 Paste 操作来获得 'AA'。
>第 3 步, 使用 Paste 操作来获得 'AAA'。
>
>```java
>public int minSteps(int n) {
>    int[] dp = new int[n + 1];
>    int h = (int) Math.sqrt(n);
>    for (int i = 2; i <= n; i++) {
>        dp[i] = i;
>        for (int j = 2; j <= h; j++) {
>            if (i % j == 0) {
>                dp[i] = dp[j] + dp[i / j];
>                break;
>            }
>        }
>    }
>    return dp[n];
>}
>```
>
>

## 其它

> 可以用动态规划解决的问题具有如下特征：
>
> - 求最优解问题（最大值和最小值）
>
> - 求可行性（True 或 False）
>
> - 求方案总数
>
> - 数据结构不可排序（Unsortable）---最小K个数
>
> - 算法不可使用交换（Non-swappable）---八皇后、全排列
>
>   ```
>   525、53、560、152、238、724、1477、713、1352、801、673、300、1143、115、940、1425、121、122、309、714、123、188、873、1027、1055、368、413、91、639、338、801、583、32、132、871、818、120、64、221、931、343、85、363
>   
>   区间问题
>   5、647、1000、516、1147、730、1312、312、546、1039
>   
>   背包问题
>   5、647、1000、516、1147、730、1312、312、546、1039
>   
>   方案总数问题
>   62、63、96、95、1155、940
>   
>   复杂问题
>   887、1067、600、1012
>   ```
>
>   

# 双指针

# 贪心



# 二分查找



# 滑动窗口

# 位运算

# 递归&分治

# 剪枝&回溯

# 堆

# 单调栈

# 排序算法

# set&map

# 栈

# 队列

# 数组

# 字符串

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

- [LeetCode\] Valid Palindrome 验证回文字符串](https://www.cnblogs.com/grandyang/p/4030114.html)

  - "A man, a plan, a canal: Panama"是回文字符串`"race a car"` 不是回文字符串

  - 给定一个字符串，判断是否为回文字符串

  - 使用双指针，遇到不是字符串的跳过即可-

# 字典树

# 并查集

# 其他类型题

