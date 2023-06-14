---
title: "N Queen Problem"
date: 2023-06-14T20:13:52+08:00
draft: true
tags: ["Algorithm","Data Structure", "Backtracking"]
---

## Purpose
TODO

## Example from LeetCode 51. N-Queens
[problem link](https://leetcode.com/problems/n-queens/description/)

### Implementation
Backtracking all possible placement, check if it is queen attack range.
```python3
class Solution:
    def solveNQueens(self, n: int) -> List[List[str]]:
        place = [-1 for _ in range(n)]
        row = 0
        place[0] = 0
        ans = []
        while True:
            if self.is_valid(place, n) is True:
                if row == n - 1:
                    ans.append([''.join(b for b in self.board[i])
                            for i in range(n)])
                else:
                    row += 1
            for j in range(row, -1, -1):
                if place[j] == n - 1:
                    place[j] = -1
                    row -= 1
                else:
                    place[j] += 1
                    break
                if j == 0 and place[j] == -1:
                    return ans
            else:
                place[row] += 1

    def is_valid(self, place, n)->bool:
        self.board = [['.' for _ in range(n)] for _ in range(n)]
        for i in range(n):
            if place[i] != -1:
                self.board[i][place[i]] = 'Q'
        # row and col
        for i in range(n):
            for j in range(i+1, n):
                if place[j] != -1 and place[i] == place[j]:
                    return False
        # cross
        for i in range(n):
            if place[i] == -1:
                continue
            for j in range(n):
                if j == 0:
                    continue
                o = i + j
                p = place[i] + j
                if  o >= 0 and o < n and \
                    p >= 0 and p < n:
                    if self.board[o][p] == 'Q':
                        return False
                o = i + j
                p = place[i] - j
                if  o >= 0 and o < n and \
                    p >= 0 and p < n:
                    if self.board[o][p] == 'Q':
                        return False
                o = i - j
                p = place[i] + j
                if  o >= 0 and o < n and \
                    p >= 0 and p < n:
                    if self.board[o][p] == 'Q':
                        return False
                o = i - j
                p = place[i] - j
                if  o >= 0 and o < n and \
                    p >= 0 and p < n:
                    if self.board[o][p] == 'Q':
                        return False
        return True
```
