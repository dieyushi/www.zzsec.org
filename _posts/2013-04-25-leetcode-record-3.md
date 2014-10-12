---
layout: post
title: "leetcode刷题记录3"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: true
disqus: y
date: 2013-04-25 23:14:48 +0800
---


昨天到今天一直没静下心来好好想想算法，导致这两天就做了一道题，太惭愧了。不过题目难度为5,想了好久的算法啊。

### Median of Two Sorted Arrays

- 题目链接：[http://leetcode.com/onlinejudge#question_4](http://leetcode.com/onlinejudge#question_4)
- 解题思路：题目求两个以排序队列的中位数，可以看成求两个队列中第k小的数。

#### Find the k-th smallest number

- 解题思路：
    - 有k个比目标更小的数，将k分成两份，一份在A的前端，一份在B的前端，设`k = i + j`
    - 如果`A[i-1] > B[j-1]`,说明B前端的j个数不可能出现我们要找的数。
    - 同理，如果`A[i-1] < B[j-1]`,说明A前端的i个数不可能出现我们要找的数。
    - 如果`A[i-1] == B[j-1]`,那么`A[i-1]`或者`B[j-1]`就是我们要找的。
    - 考虑边界情况。如`k==1`, `m==0`。
    - 重复以上步骤，直到返回。

<!--more-->

- 实现：

```cpp
int findKthSmallest(int A[], int m, int B[], int n, int k) {
    if (m > n)
        return findKthSmallest(B, n, A, m, k);
    if (m == 0)
        return B[k-1];
    if (k == 1)
        return min(A[0], B[0]);

    int i = min(k/2, m), j = k - i;
    if (A[i-1] < B[j-1])
        return findKthSmallest(A+i, m-i, B, n, k-i);
    else if (A[i-1] > B[j-1])
        return findKthSmallest(A, m, B+j, n-j, k-j);
    else
        return A[i-1];
}
```

#### Find the Median

关于中位数的定义可以参考([这里](http://en.wikipedia.org/wiki/Median))。

- 实现：

```cpp
double findMedianSortedArrays(int A[], int m, int B[], int n) {
    int k = (m + n) /2;
    if ((m+n)%2)
        return findKthSmallest(A,m,B,n,k+1);
    else
        return (findKthSmallest(A,m,B,n,k)+findKthSmallest(A,m,B,n,k+1))/2.0;
}
```

### 参考

Leetcode博客上提供了更好的方法([链接](http://leetcode.com/2011/03/median-of-two-sorted-arrays.html))，可以参考。
