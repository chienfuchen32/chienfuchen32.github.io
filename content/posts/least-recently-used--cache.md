---
title: "Least Recently Used  Cache"
date: 2023-05-31T20:10:53+08:00
draft: true
tags: ["Algorithm","Data Structure"]
---

## Purpose
TODO

## Example from LeetCode 146. LRU Cache
[problem link](https://leetcode.com/problems/lru-cache/)

### Implementation
TODO commemt
```python3
class Node:
    def __init__(self, key, value, prev=None, nxt=None):
        self.key = key
        self.value = value
        self.prev = prev
        self.next = nxt

class DoubleLinkList:
    def __init__(self):
        self.head = None
        self.tail = None
        self.length = 0

    def add_node_to_tail(self, node: Node) -> None:
        self.length += 1
        if self.head is None:
            self.head = node
            return 
        if self.tail is None:
            self.tail = node
            self.tail.prev = self.head
            self.head.next = self.tail
            return
        self.tail.next = node
        node.prev = self.tail
        self.tail = node

    def remove_node_from_head(self) -> int:
        self.length -= 1
        old_key = self.head.key
        self.head = self.head.next
        if self.head is not None:
            self.head.prev = None
        return old_key

    def remove_node_from_node(self, node) -> None:
        self.length -= 1
        if node.prev is not None:
            node.prev.next = node.next
        if node.next is not None:
            node.next.prev = node.prev
        if node == self.tail:
            if self.tail.prev is not None:
                self.tail.prev.next = None
            self.tail = self.tail.prev
        if node == self.head:
            if self.head.next is not None:
                self.head.next.prev = None
            self.head = self.head.next

    def print_all_node(self) -> None:
        ptr = self.head
        while ptr is not None:
            ptr = ptr.next

class LRUCache:
    def __init__(self, capacity: int):
        self.cache = {} # {key: Node}
        self.link_list = DoubleLinkList()
        self.capacity = capacity

    def get(self, key: int) -> int:
        if self.cache.get(key, None) is not None:
            node = self.cache[key]
            value = node.value
            new_node = Node(key, value)
            self.link_list.remove_node_from_node(node)
            self.cache[key] = new_node
            self.link_list.add_node_to_tail(new_node)
            return value
        return -1


    def put(self, key: int, value: int) -> None:
        if self.cache.get(key, None) is not None:
            node = self.cache[key]
            del self.cache[key]
            self.link_list.remove_node_from_node(node)
        else:
            if self.link_list.length >= self.capacity:
                head_key = self.link_list.remove_node_from_head()
                del self.cache[head_key]
        new_node = Node(key, value)
        self.cache[key] = new_node
        self.link_list.add_node_to_tail(new_node)
```
