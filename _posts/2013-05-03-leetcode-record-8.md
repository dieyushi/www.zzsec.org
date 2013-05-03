---
layout: post
title: "leetcode刷题记录8"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: 
---
{% include JB/setup %}

### 前言

这几天出差，打乱了自己做题的节奏，也只能抽空做几道题来保证做题的感觉了。

### 题目

#### Swap Nodes in Pairs

- 题目链接：http://leetcode.com/onlinejudge#question_24
- 解题思路：知道了当前`ListNode`的前一个和后一个，这道题就是单纯的链表操作了。
- 实现：

<!--more-->

```cpp
ListNode *swapPairs(ListNode *head) {
    ListNode *curr = head, *tmp = 0, *next = 0, *pre = 0;
    if (curr && curr->next)
        head = curr->next;
    while(curr) {
        if (curr->next == 0) return head;
        next = curr->next;
        if (pre != 0) pre->next = next;
        tmp = next->next;
        next->next = curr;
        curr->next = tmp;
        pre = curr;
        curr = curr->next;
    }
    return head;
}
```

#### Remove Duplicates from Sorted Array

- 题目链接：http://leetcode.com/onlinejudge#question_26
- 解题思路：遍历一遍，每次有不同的值就保存
- 实现：

```cpp
int removeDuplicates(int A[], int n) {
    if (n == 0) return 0;
    int start = 0;
    int curr = A[0];

    for (int i=0;i<n;i++){
        if (A[i] != curr) {
            A[start++] = curr;
            curr = A[i];
        }
    }
    A[start++] = curr;
    return start;
}
```

#### Remove Element

- 题目链接：http://leetcode.com/onlinejudge#question_27
- 解题思路：和上题类似，很简单的题目。
- 实现：

```cpp
int removeElement(int A[], int n, int elem) {
    if (n==0) return 0;
    int start = 0;

    for (int i=0;i<n;i++){
        if (A[i] != elem) {
            A[start++] = A[i];
        }
    }
    return start;
}
```
