# 题目

给定一棵二叉搜索树，请找出其中第k大的节点。

# 解题思路

本文解法基于此性质：二叉搜索树的中序遍历为 **递增序列** 。

根据以上性质，易得二叉搜索树的 中序遍历倒序 为 递减序列 。
因此，求 “二叉搜索树第 k 大的节点” 可转化为求 “此树的中序遍历倒序的第 k 个节点”。

# 代码

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
    int res, k;
    public int kthLargest(TreeNode root, int k) {
        this.k = k;
        dfs(root);
        return res;
    }
    void dfs(TreeNode root) {
        if(root == null) return;
        dfs(root.right);
        k=k-1;
        if(k == 0) {
            res = root.val;
            return ;
        }
        dfs(root.left);
    }
}



```

