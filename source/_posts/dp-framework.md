---
tags: dynamic programming, python, algorithm
title: Review of dynamic programming framework (fibonnaci, coin change)
---

[![hackmd-github-sync-badge](https://hackmd.io/6MGw9QgbTe-z_5tI3kSwfg/badge)](https://hackmd.io/6MGw9QgbTe-z_5tI3kSwfg)

## Introduction

This is review for dynamic programming framework [動態規劃解題套路框架](https://labuladong.github.io/algo/1/4/)

Three elements of DP (dynamic programming):
- base case: The base case that the answer is known.
- subproblems: Child problems such that once they are sovled we can determine unique solution for their parent; subproblems are independent from each others meaning the optimal solution of each subproblem won't affect its peer. it is often express through state transition equation.
- memory table: To reduce duplicate calculation

## Implementation
[Leetcode 509: Fibonacci number](https://leetcode.com/problems/fibonacci-number/)

- base case: when n is 0 or 1, we harded the answer to be 0 or 1.
- state transition equation: f(n) = f(n-1) + f(n-2)
- memory table: two elements

- space complexity: O(2)
- time complexity: O(n)
```python=
class Solution:
    def fib(self, n: int) -> int:
        if n == 0 or n == 1:
            return n
        dp_1 = 0
        dp_2 = 1
        dp_new = None
        for i in range(2,n+1):
            dp_new = dp_1 + dp_2
            dp_1 = dp_2
            dp_2 = dp_new
        return dp_new
```

- Base case: 0 amount requires 0 coin
- Subproblem: Assume there are k type of coins, DP(amount, coins) = 1 + min(DP(amount-coin[0],coins),DP(amount-coin[1],coins), ..., DP(amount-coin[k-1],coins))
- memory table: A list with length `amount`+1 with its elements initialised to amount + 1 so that if the number is not changed it means the subproblem is not solved yet hence we need to calculate the optimal solution for the subproblem and record the result; otherwise retrieve the result

- time complexity: O(k*n) because with memory, the number of subproblem is at most n and each subproblem we loop through k coins.
- space complexity: O(n)

[Leetcode 322: Coin change](https://leetcode.com/problems/coin-change/)
```python=
class Solution:
    def coinChange(self, coins: List[int], amount: int) -> int:
        self.dp = [ amount+1 for i in range(amount+1)]
        self.amount = amount
        self.recursion(coins, amount)
        if self.dp[amount] == self.amount+1:
            return -1
        else:
            return self.dp[amount]
    def recursion(self,coins,amount):
        
        if amount == 0:
            self.dp[0]=0
            return 0
        if amount<min(coins):
            return -1
        if self.dp[amount] !=  self.amount+1:
            return self.dp[amount]
        
        for coin in coins:
            if coin>amount:
                continue
            if self.dp[amount-coin] != self.amount+1:
                sub_problem = self.dp[amount-coin]
            else:
                sub_problem = self.recursion(coins,amount-coin)
            if sub_problem + 1 < self.dp[amount] and sub_problem >= 0:
                self.dp[amount] = sub_problem + 1

        if self.dp[amount] !=  self.amount+1:
            return self.dp[amount]
        else:
            self.dp[amount] = -1
            return -1     
```