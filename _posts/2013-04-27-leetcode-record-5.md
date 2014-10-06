---
layout: post
title: "leetcode刷题记录5"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: true
disqus: y
date: 2013-04-27 23:02:09 +0800
---


### 前言

现在发现能找到一段时间能安心的做自己的事情不被别人打扰是多么的舒服，现在终于有点时间记录下今天抽空理解的一个题目`Regular Expression Matching`,一开始看到题目难度为5,直接跳过，今天思考了下，能够理解怎么做的了，记录下～


### 题目

#### Regular Expression Matching

<!--more-->

- 题目链接：http://leetcode.com/onlinejudge#question_10
- 题目思路：
    - 字符串`const char *s`和用于匹配的`pattern`，`const char *p`。我们需要关注的是`p+1`。
    - 如果`*(p+1)`不是`'*'`,那么只有`*p == *s`或者`*p == '.'`(这时候`*s`不能为`\0`)的时候才可能继续匹配。
    - 如果`*(p+1)`为`'*'`,对于`s`，依次比较`*s`和`*(p+2)`,直到匹配或者不匹配。
- 实现：

```cpp
bool isMatch(const char *s, const char *p) {
    if (*p == '\0') return *s == '\0';
 
    if (*(p+1) != '*') {
        if (*p == *s || *p == '.' && (*s) != '\0') return isMatch(s+1, p+1);
        return false;
    } else {
        while (*p == *s || *p == '.' && (*s) != '\0') {
            if (isMatch(s, p+2)) return true;
            s++;
        }
        return isMatch(s, p+2);
    }
}
```
