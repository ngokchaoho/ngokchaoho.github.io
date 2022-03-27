---
title: Review of Back tracking (one kind of DFS) framework
tag: DFS, backtrack, python
---
## Introduction

This is a review of [回溯算法解題套路框架](https://labuladong.github.io/algo/1/5/)
Framework:
```psesudo=
if path meets requirement:
    result.add(path.copy())
for option in available options:
    path.add(option) # make a decision
    backtrack(path,available options)
    path.remove(option) # cancle a decision
```

[Leet coce 46. Permutations](https://leetcode.com/problems/permutations/)
Using the general framework, it is perhap easier to find this solution first, which used addtional time complexity to find if the options is used.

time complexity: O( n!^2 )
space complexity: O( (n!+1)*n )
```python=
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        self.result=[]
        self.recursive([],nums)
        return self.result
    def recursive(self,track,nums):
        if len(track) == len(nums):
            self.result.append(track.copy())
            return
        for element in nums:
            if element in track:
                continue
            track.append(element)
            self.recursive(track,nums)
            track.pop()
```

but through swapping we don't need to use extra time to check if elements have been used before; elements in current (start,end) range hasn't been used before.
time complexity: O(n!)
space complexity: O( (n!+1)*n )
```python=
class Solution:
    def permute(self, nums: List[int]) -> List[List[int]]:
        self.result=[]
        self.recursive(0,len(nums)-1,nums)
        return self.result
    def recursive(self,start,end,nums):
        if start == end:
            self.result.append(nums.copy())
            return
        for i in range(start,end+1):
            temp = nums[start]
            nums[start] = nums[i]
            nums[i] = temp
            self.recursive(start+1,end,nums)
            temp = nums[i]
            nums[i] = nums[start]
            nums[start] = temp
```

[Leetcode 51. N-Queens](https://leetcode.com/problems/n-queens/)

Time complexity: [O(n!)](https://stackoverflow.com/questions/21059422/time-complexity-of-n-queen-using-backtracking) since T(n) = n * T(n-1) + O(n^2) expand to T(n) = O(n!) + O(n^3) = O(n!)

Space complexity: O(n^4)

```python=
from copy import deepcopy
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        self.board = [['.' for j in range(n)] for i in range(n)]
        self.result = []
        self.n = n 
        self.backtrack(0)
        return self.result
    def is_valid(self, i, j):
        for x in range(i):
            # check same column
            if self.board[x][j] == 'Q':
                return False
            if j-(i-x) >= 0 and self.board[x][j-(i-x)] == 'Q':
                return False
            if j+(i-x) < self.n and self.board[x][j+(i-x)] == 'Q':
                return False
        return True
    def backtrack(self,i):
        if i == self.n:
            temp = []
            for x in range(self.n):
                temp.append("".join(self.board[x]))
            self.result.append(temp)
            return
        for j in range(self.n):
            if not self.is_valid(i,j):
                continue
            self.board[i][j] = 'Q'
            self.backtrack(i+1)
            self.board[i][j] = '.'
```
        
