---
layout: post
title: "leetcode刷题记录1"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: 
---
{% include JB/setup %}

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
- 解题思路：这个题彻彻底底的把我伤透了。。一开始题目理解错了，看成了一个是正向，一个是逆向链表，结果是正向的，结果一提交，只有1/4的Pass，看了半天代码都不知道什么问题。。最后重新读了读题目，仔细看看了测试用例才发现错了，结果也造成了我采用的方法是先把这两个变成数字，相加后再转成链表，可想而知，大数据的测试让我的int直接就maxvalue了。。改成long long类型的，很丑陋的Pass了。
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

## 实验室又赶人了。。未完待续
