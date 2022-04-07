---
Tags: binary tree
Title: Review of labuladong's introduction of binary tree 
---
## Introduction
This a review of [東哥帶你刷二叉樹](https://labuladong.github.io/algo/1/7/)

## Framework
You can solve problem with binary tree with two methods:
- backtracking
- dynamic programming

Using the [framework of backtracking](https://hackmd.io/3HEM3FA1RiKcWVJhSkC6ww) we make decision preorder and cancel decision postorder:
[Leet code 104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/)
Time complexity: O(N) traverse through all nodes
Space complexity: O(N) worst case the callback stack depth would be N
```python=
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        self.depth = 0
        self.result = 0
        self.traverse(root)
        return self.result
    def traverse(self, node):
        if node is None:
            if self.depth>self.result:
                self.result = self.depth
            return
        self.depth += 1 # preorder position
        self.traverse(node.left)
        self.traverse(node.right)
        self.depth -= 1 # postorder position
```
Using the [framework of dynamic programming](https://hackmd.io/6MGw9QgbTe-z_5tI3kSwfg), we define subproblem and obtain optimal solution for each subproblem via state transition function `max(left_max,right_max)+1`

Time complexity: O(N) traverse through all nodes
Space complexity: O(N) worst case the callback stack depth would be N
```python=
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0
        left_max = self.maxDepth(root.left)
        right_max = self.maxDepth(root.right)
        return max(left_max,right_max)+1
```

[leetcode 543. Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/)
Time complexity: O(N) traverse through all nodes
Space complexity: O(N) worst case the callback stack depth would be N

Which is just an extension of above, if you discover that the subproblem's solution equal to `current_max_diameter = max_left_depth + max_right_depth` and in postorder we already have `max_left_depth` and `max_right_depth`
```python=
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def diameterOfBinaryTree(self, root: Optional[TreeNode]) -> int:
        self.max_diameter = 0
        self.max_depth(root)
        return self.max_diameter
    def max_depth(self,root):
        if root is None:
            return 0
        right_depth = self.max_depth(root.right)
        left_depth = self.max_depth(root.left)
        
        my_diameter = left_depth + right_depth
        if self.max_diameter < my_diameter:
            self.max_diameter = my_diameter 
        return max(right_depth,left_depth)+1
    
        
```
[144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/)
Typical solution from definition
time complexity: O(N)
```python=
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        self.res = []
        self.traverse(root)
        return self.res
    def traverse(self,node):
        if node:
            self.res.append(node.val)
            self.traverse(node.left)
            self.traverse(node.right)
```
Using the recursive kind of thinking, we can also divide the preorder traversal problem to subproblems of its sub-trees:

But the time complexity is O(N^2) due to python's list extend is O(N) and we do that to every node.
```python=
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        res = []
        if root is None:
            return res
        else:
            res.append(root.val)
            res.extend(self.preorderTraversal(root.left))
            res.extend(self.preorderTraversal(root.right))
            return res
```