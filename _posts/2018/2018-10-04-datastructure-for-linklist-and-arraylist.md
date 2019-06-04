---
title: 数组、链表对比及应用
date: 2018-10-04 18:46:58
tags: ['Java', '数据结构']
category: datastructureandalgo
---

## 1. 数组和链表的区别
- 1.1 底层存储结构
	- 数组需要一块连续的内存空间进行存储
	- 链表通过“指针”将一组零散的内存块串联起来使用
- 1.2 性能
	- 链表和数组的（增删查）时间复杂度正好相反
	- 数组使用连续的内存空间，可以借助缓存机制提高效率；链表不连续，所以无法借助缓存机制
	- 数组大小固定，当需要申请更大的空间，需要拷贝数据，很耗时；而链表纯天然的支持动态扩容
	- 相对来说，链表比较耗内存，因为需要记录节点指针，内存消耗翻倍

【注】和数组相比，链表更适合插入、删除操作频繁的场景，查询的时间复杂度较高。不过，在具体的软件开发中，要对数组和链表的各种性能进行对比，综合考虑再决定使用两者中的哪一个。

## 2. 基于单链表的LRU缓存
### 2.1 什么是缓存？
缓存是一种提高数据读取性性能的技术，在硬件设计、软件开发中都有着广泛的应用，比如常见的CPU缓存、数据库缓存、浏览器缓存等。

### 2.2 缓存淘汰策略有哪些？
Q：缓存的特点是什么？
A：缓存的大小是有限的，当缓存被用满时，哪些数据应该被清理出去，哪些数据应该被保留下来。这时就需要用到缓存淘汰策略。

常见的3种缓存淘汰策略：先进先出策略（**FIFO**，First In, First Out），最少使用策略（**LFU**，Least Frequently Used），最近最少使用策略（**LRU**，Least Recently Used）

### 2.3 单链表实现LRU缓存淘汰策略
当访问的数据没有存储在缓存的链表中时，直接将数据插入链表表头，时间复杂度为O(1)；
当访问的数据存在于存储的缓存的链表中，将该数据对应的节点，移动到链表表头，时间复杂度为O(n)；
如果缓存被占满，则从链表尾部的数据开始清理，时间复杂度为O(1)。

[实现源码请点击这里查看](https://github.com/Zychaowill/AlgorithmAndDataStructure/blob/master/src/main/java/zychaowill/datastructure/basic/list/LRUCache.java)

## 3. 判断单链表是否为回文链表
### 3.1 定位链表中间节点
Q：在不知道链表长度或者节点个数的情况，如何定位链表中间节点呢？
A：我们可以通过快慢指针进行定位。慢指针每次前进一步，快指针每次前进两步。

```Java
	private <T> Node<T> shootingMiddleNode(Node<T> node) {
		if (node == null || node.next == null) {
			return node;
		}
		Node<T> slow = node;
		Node<T> fast = node;
		while (slow != null && fast.next != null && fast.next.next != null) {
			slow = slow.next;
			fast = fast.next.next;
		}
		return slow;
	}
```

### 3.2 从中间节点下一个节点开始进行翻转链表
```Java
	private <T> Node<T> reverseList(Node<T> root) {
		Node<T> pre = null;
		Node<T> next = null;
		while (root != null) {
			next = root.next;
			root.next = pre;
			pre = root;
			root = next;
		}
		return pre;
	}
```

### 3.3 比对是否为回文
```Java
	public <T> boolean isPalindrome(Node<T> root) {
		if (root == null || root.next == null) {
			return true;
		}
		Node<T> middleNode = shootingMiddleNode(root);
		middleNode.next = reverseList(middleNode.next);

		Node<T> first = root;
		Node<T> second = middleNode.next;
		while (first != null && second != null && Objects.equals(first.data, second.data)) {
			first = first.next;
			second = second.next;
		}
		return second == null;
	}
```

[完整代码实现可以点击这里查看](https://github.com/Zychaowill/AlgorithmAndDataStructure/blob/master/src/main/java/zychaowill/datastructure/basic/list/PalindromeListChecker.java)
当然，这里通过双向链表进行实现会简单很多，毕竟存在前驱和后继指针。

【注】除了单链表外，还有演化出来的双向链表，循环单链表，循环双向链表。
