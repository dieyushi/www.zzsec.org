---
layout: post
title: "leetcode刷题记录4"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: 
---
{% include JB/setup %}

### 前言

今天事情好多，到最后终于可以静下心来好好想想题目了。最后终于理解了怎么用O(n)的时间复杂度求最长回文字串。记录一下。

一些题外话，一直用着`redcarpet`来渲染markdown，却一直没有配置。。今天把extensions加上了，可以用表格和`autolink`了～

```yaml
redcarpet:
    extensions: ["no_intra_emphasis", "fenced_code_blocks", "autolink", "tables", "with_toc_data"]
```

### 题目

#### Longest Palindromic Substring

- 题目链接：http://leetcode.com/onlinejudge#question_5
- 解题思路：
    - 首先用一个很巧妙的方法将所有可能的奇数/偶数长度的回文字符串都变成了奇数长度。方法是在每个字符两边都插入一个特殊符号。为了不要处理越界问题，在字符串开始是插入另一个特殊字符。如`abbabcdcba`就变成了`$#a#b#b#a#b#c#d#c#b#a#`。
    - 用一个数组`p[i]`来记录以`s[i]`为中心的最长回文字符串向左/右扩张的最大长度(包括`s[i]`)，例子如下：

    S | $ | # | a | # | b | # | b | # | a | # | b | # | c | # | d | # | c | # | b | # | a | #
    ---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---
    P | 0 | 1 | 2 | 1 | 2 | 5 | 2 | 1 | 4 | 1 | 2 | 1 | 2 | 1 | 8 | 1 | 2 | 1 | 2 | 1 | 2 | 1

<!--more-->

    - 求`p[i]`数组中最大值`maxlen`，`maxlen-1`就是原字符串中最长回文字符串的长度。
    - `p[i] = min(p[2*id-i], mx-i)` 其中id为最大回文字串中心位置，`mx为id+p[id]`,即最大回文字串的边界。式子可以这样推导出来：
        - 当`mx-i > P[2*id - i]`时，由于`i和2*id-i对称`，`p[i] = p[j]`
        - 当`mx-i < P[2*id - i]`时，同样由于对称性，`i + p[i] >= mx-i`
        - 当`mx > i`时，`p[i]`可以从`min(p[2*id-i], mx-i)`开始向两侧匹配。
        - 当`mx <= i`时，不能推断具体情况，只能从`p[i] = 1`开始，一对一对的匹配。
    - 最后的字串范围为`[(centerIndex + 1 - maxlen)/2, maxlen-1]`, `centerIndex`为`maxlen`对应字符的位置。

- 实现：

```cpp
string longestPalindrome(string s) {
    string stemp = "^";
    for (int i=0; i < s.length(); i++)
        stemp = stemp + "#" + s[i];
    stemp = stemp + "#";
    if (s.empty())
        stemp = "^$";
 
    int mx = 0, id = 0;
    int p[2002] = {0};
    for(int i=1; i<stemp.length(); i++){
        p[i] = mx > i ? min(p[2*id-i], mx-i) : 1;
        while (stemp[i+p[i]] == stemp[i-p[i]])
            p[i]++;
        if (i+p[i] > mx) {
            mx = i + p[i];
            id = i;
        }
    }
 
    int maxlen = 0;
    int centerIndex = 0;
    for (int i = 1; i < stemp.length(); i++) {
        if (p[i] > maxlen) {
            maxlen = p[i];
            centerIndex = i;
        }
    }
 
    return s.substr((centerIndex + 1 - maxlen)/2, maxlen-1);
}
```

