---
layout: post
title: "leetcode刷题记录7"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: true
comments: true
date: 2013-04-29 20:45:29 +0800
---
{% include JB/setup %}

### 前言

这几天要抓紧时间写别的程序，要在出差前搞定，只能抽空做两道题了。难度都不高，写起来注意点还是可以一次就pass的。

#### Generate Parentheses

- 题目链接：http://leetcode.com/onlinejudge#question_22
- 解题思路：
    - 当`n == 1`时，返回`"()"`
    - 当`n > 1`时， 先计算`generateParenthesis(n-1)`,对返回的vector中每一个string进行操作。我们可以新增一对`()`，放在原来里面的任何一位后面都可以。使用`set`来保存唯一的值。
- 实现：

<!--more-->

```cpp
vector<string> generateParenthesis(int n) {
    vector<string> ret;
    if (n == 0) {
        ret.push_back("");
        return ret;
    }
    if (n == 1){
        ret.push_back("()");
        return ret;
    }
    set<string> ret1;

    vector<string> last = generateParenthesis(n-1);
    for (auto iter = last.begin(); iter != last.end(); ++iter) {
        int len = iter->size();
        for (int i=0; i<len; ++i) {
            string tmp = *iter;
            tmp.insert(i,string("()"));
            ret1.insert(tmp);
        }
    }

    for (auto iter = ret1.begin(); iter != ret1.end(); ++iter) {
        ret.push_back(*iter);
    }
    return ret;
}
```

#### Merge Two Sorted Lists

- 题目链接：http://leetcode.com/onlinejudge#question_21
- 解题思路：两个链表逐位比较就行，没什么难度。
- 实现：

```cpp
ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
    ListNode *ret = new ListNode(0);
    ListNode *tmp = ret;
    while (l1 != 0 && l2 != 0) {
        if (l1->val < l2->val) {
            tmp->next = l1;
            l1 = l1->next;
        } else {
            tmp->next = l2;
            l2 = l2->next;
        }
        tmp = tmp->next;
    }
    if (l1) {
        tmp->next = l1;
    }
    if (l2) {
        tmp->next = l2;
    }
    ret = ret->next;
    return ret;
}
```

#### Merge k Sorted Lists

- 题目链接：http://leetcode.com/onlinejudge#question_23
- 解题思路：使用上一个题目`Merge Two Sorted Lists`的结果，两两merge。

```cpp
ListNode *mergeKLists(vector<ListNode *> &lists) {
    if (lists.size() == 0) return 0;
    ListNode *p = lists[0];
    for (int i=1; i<lists.size(); i++) {
        p = mergeTwoLists(p, lists[i]);
    }
    return p;
}
```
