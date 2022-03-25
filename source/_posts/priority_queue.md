---
tags: Python, Algorithm, max-heap
---
---
title: Simple Python Implementation of Priority Queue via Max heap
---
[![hackmd-github-sync-badge](https://hackmd.io/qDlvP3e5TXSB1Mm2ZMy2OQ/badge)](https://hackmd.io/qDlvP3e5TXSB1Mm2ZMy2OQ)



While python has heapq to use (which is min heap), e.g.


>heapq.heappush(heap, item)
 >   Push the value item onto the heap, maintaining the heap invariant.

> heapq.heappop(heap)
>    Pop and return the smallest item from the heap, maintaining the heap invariant. If the heap is empty, IndexError is raised. To access the smallest item without popping it, use heap[0].


Just as a practice, I try to write to write a max heap from scratch for integer

```python=
class PriorityQueue:
    def __init__(self):
        self._heap = [None,]
        self._heap_size = 0
    def exch(self,A,B):
        temp = self._heap[A]
        self._heap[A] = self._heap[B]
        self._heap[B] = temp
    def _swim(self):
        k = self._heap_size
        parent = k // 2
        while parent>0 and self._heap[parent]<self._heap[k]:
            self.exch(parent,k)
            k = parent
            parent = k // 2
    def _sink(self):

            k = 1
            left_child = k*2
            right_child = k*2+1

            while left_child<=self._heap_size:
                bigger = left_child
                if k*2+1 <= self._heap_size and self._heap[left_child]<self._heap[right_child]:
                        bigger = right_child
                
                if self._heap[k]<self._heap[bigger]:
                    self.exch(k,bigger)
                k = bigger
                left_child = k*2
                right_child = k*2+1

    def push(self,element):
        self._heap.append(element)
        self._heap_size += 1
        self._swim()
    def pop(self):
        max = self._heap[1]
        self._heap[1] = self._heap[self._heap_size]
        self._heap[self._heap_size] = None
        self._heap_size -= 1
        self._sink()
        return max
        

from random import randint

pq = PriorityQueue()
for  i in range(10):
    n = randint(1,10)
    pq.push(n)

result = []
for i in range(10):
    result.append(pq.pop())
print(result)
```