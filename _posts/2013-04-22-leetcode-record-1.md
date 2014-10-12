---
layout: post
title: "leetcode刷题记录1"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: true
disqus: y
date: 2013-04-22 23:12:14 +0800
---


### 前言

转眼间2013年都要进入5月了，马上就进入毕业季了，开始准备笔试+面试，看到很多人推荐[leetcode](http://leetcode.com/onlinejudge),我也准备去刷一边上面的题目，复习下算法的知识。很长一段时间以来，写程序都用`Python`或`Golang`，重新使用`C++`还真有点不习惯。整整一下午加晚上就完成了四道题。代码惨不忍睹啊，看来想要写出bug free的代码还是任重道远啊。

### 题目

回到题目本身，`leetcode`的难度不算特别大，有很大一部分比较简单的题目，按照这个网站([链接](http://leetcode.cloudfoundry.com/))的整理，第一天我只挑选了较为简答的难度为3的题目（难度区间1-5）。下面记录下我的解题思路。

<!--more-->

#### Two Sum

- 题目链接：[http://leetcode.com/onlinejudge#question_1](http://leetcode.com/onlinejudge#question_1)
- 解题思路：遍历vector，使用std::find查找target-*iter是否在vector中。
- 实现：

```cpp
vector<int> twoSum(vector<int> &numbers, int target) {
    vector<int> ret;
    for (auto iter = numbers.begin(); iter != numbers.end(); ++iter) {
        int another = target - *iter;
        auto iter1 =std::find(iter+1, numbers.end(), another);
        if (iter1 != numbers.end()) {
            if(iter <= iter1) {
                ret.push_back(iter-numbers.begin()+1);
                ret.push_back(iter1-numbers.begin()+1);
            }else{
                ret.push_back(iter1-numbers.begin()+1);
                ret.push_back(iter-numbers.begin()+1);
            }
            return ret;
        }
    }
    return ret;
}
```

#### Add Two Numbers

- 题目链接：[http://leetcode.com/onlinejudge#question_2](http://leetcode.com/onlinejudge#question_2)
- 解题思路：这个题彻彻底底的把我伤透了。。一开始题目理解错了，看成了一个是正向，一个是逆向链表，结果是正向的，结果一提交，只有1/4的Pass，看了半天代码都不知道什么问题。。最后重新读了读题目，仔细看看了测试用例才发现错了，结果也造成了我采用的方法是先把这两个变成数字，相加后再转成链表，可想而知，大集合的测试让我的int直接就maxvalue了。。改成long long类型的，很丑陋的Pass了。
- 实现：

```cpp
ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
    long long sum1 = 0, sum2 = 0;
    for (int i=0; l1; l1 = l1->next) {
        sum1 = sum1 + l1->val * std::pow(10,i);
        i++;
    }

    for (int i=0; l2; l2 = l2->next) {
        sum2 = sum2 + l2->val * std::pow(10,i);
        i++;
    }

    long long sum = sum1 + sum2;
    ListNode *l3 = 0;
    ListNode *tail = 0;
    if (sum == 0){
        l3 = new ListNode(0);
        return l3;
    }
    while (sum) {
        long long rem = sum%10;
        sum = sum/10;
        ListNode *temp = new ListNode(rem);
        if (l3) {
            tail->next = temp;
            tail = temp;
        } else {
            l3 = temp;
            tail = temp;
        }
    }
    return l3;
}
```

#### Longest Substring Without Repeating Characters

- 题目链接：[http://leetcode.com/onlinejudge#question_3](http://leetcode.com/onlinejudge#question_3)
- 解题思路：建一个ascii表的bool数组，从字符串的第i位开始，将对应出现的设置为true，直到出现重复的字母，位置j，j-i就可能是一个最大子字符串。
- 实现：

```cpp
int GetLongestSubstr(string s)
{
    int n = s.length();
    int i=0,j=0;
    int maxLen = 0;
    bool exist[256] = {false};

    while (j<n) {
        if (exist[s[j]]) {
            maxLen = max(maxLen, j-i);
            while (s[i] != s[j]) {
                exist[s[i]] = false;
                i++;
            }
            i++;
            j++;
        } else {
            exist[s[j]] = true;
            j++;
        }
    }
    return max(maxLen, n-i);
}
```

#### ZigZag Conversion

- 题目链接：[http://leetcode.com/onlinejudge#question_6](http://leetcode.com/onlinejudge#question_6)
- 解题思路：话说我看懂这个题目都用了很久，然后，自己画两个就知道规律了，每个的个数为2n-2,n为行数，第一行和最后一行一个字母，其余的都是两个，规律也很容易找到。不过一开始忘了考虑n=1的情形，大集合的那个没Pass。
- 实现：

```cpp
string ZigZagconvert(string s, int nRows) {
    string ret;
    int len = s.length();
    int everyItem = 2 * nRows -2;

    if (everyItem == 0) {
        return s;
    }
    int cout = len/everyItem;
    int left = len%everyItem;

    //first line
    for (int i=0; i <= (cout-1); i++) {
        ret.append(1,s[(2*nRows-2)*i]);
    }
    if (left) {
        ret.append(1,s[(2*nRows-2)*(cout)]);
    }

    // next serval lines
    for (int i=1; i <= nRows-2; i++){
        for (int j=0; j<= (cout-1); j++){
            ret.append(1,s[(2*nRows-2) * j + i]);
            ret.append(1,s[(2*nRows-2) * j + i + (2*nRows-2)-2*i]);
        }
        if ((2*nRows-2) * cout+ i <= len-1) {
            ret.append(1,s[(2*nRows-2) * cout + i]);
        }
        if ((2*nRows-2) * cout + i + (2*nRows-2)-2*i <= len-1) {
            ret.append(1,s[(2*nRows-2) * cout + i + (2*nRows-2)-2*i]);
        }
    }

    //add the last line
    for (int i=0; i <= cout-1; i++) {
        ret.append(1,s[(2*nRows-2)*i + nRows -1]);
    }

    if ((2*nRows-2)*cout + nRows -1 <= len-1) {
        ret.append(1,s[(2*nRows-2)*cout + nRows -1]);
    }

    return ret;
}
```

### 总结

- 现在使用VS来写代码，离面试时真正的纸上写代码还有很大差距。
- 第一遍写出的代码很多bug，多做练习，争取一遍bug free。
- 很多时候没有考虑特殊情况，需要继续努力。
- 刷题虽然有点累，但是很充实。
