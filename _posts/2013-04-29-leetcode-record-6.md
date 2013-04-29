---
layout: post
title: "leetcode刷题记录6"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: 
---
{% include JB/setup %}

### 前言

连续做了几道很简单的题目，都是一看就知道是什么思路的题目，重要的是怎么才能写出没有bug的程序。感觉自己直接完成的题目多少要改动下，没有写完就直接通过测试的。

### 题目

#### 3Sum

- 题目链接：http://leetcode.com/onlinejudge#question_15
- 解题思路：
    - 可以将题目转换成2Sum，给定target，求解。解决这个题目首先要将数组排序，然后left定义为最左边，right定义为最右边，如果`left + right < target`,左边的数应该更大，往右移以为，反之右边向左移，相等的话就要同时移动。这样可以找出所有的组合。
    - 按照这个思路，已排好序的数组只要固定一个即可。
- 实现：

<!--more-->

```cpp
vector<vector<int> > threeSum(vector<int> &num) {
    sort(num.begin(), num.end());
    set<vector<int> > ret;
    vector<vector<int> > ret1;
    for (auto iter = num.begin(); iter != num.end(); ++iter) {
        auto left = iter + 1, right = num.end()-1;
        while (left < right) {
            if (*iter + *left + *right > 0)
                --right;
            else if (*iter + *left + *right < 0)
                ++left;
            else {
                vector<int> tmp;
                tmp.push_back(*iter);
                tmp.push_back(*left);
                tmp.push_back(*right);
                ret.insert(tmp);
                --right;
                ++left;
            }
        }
    }
    for (auto iter = ret.begin(); iter != ret.end(); ++iter) {
        ret1.push_back(*iter);
    }
    return ret1;
}
```

### 3Sum Closest

- 题目链接：http://leetcode.com/onlinejudge#question_16
- 解题思路：这个题目和`3Sum`很像，只是每次要计算三个数的和与目标值的差的绝对值，同时再注意下比目标大还是小就可以了。
- 实现：

```cpp
int threeSumClosest(vector<int> &num, int target) {
    sort(num.begin(), num.end());
    int mindist = INT_MAX;
    bool sign = false;
    int n = num.size();
    for (int i=0; i<n; i++) {
        int j=i+1, k=n-1;
        while (j < k) {
            int sum = num[i] + num[j] + num[k];
            int orig = mindist;
            mindist = min(mindist, abs(sum - target));
            if(sum > target) {
                if (mindist < orig)
                    sign = true;
                k--;
            } else if (sum < target) {
                if (mindist < orig)
                    sign = false;
                j++;
            } else {
                return target;
            }
        }
    }
    return (sign == true) ? (mindist + target) : (target - mindist);
}
```

#### 4Sum

- 题目链接：http://leetcode.com/onlinejudge#question_18
- 解题思路：参考`3Sum`，基本一样。
- 实现：

```cpp
vector<vector<int> > fourSum(vector<int> &num, int target) {
    set<vector<int> > ret;
    sort(num.begin(), num.end());

    int n = num.size();
    for (int i=0; i<n; i++) {
        for (int j=i+1; j<n; j++) {
            int k = j + 1, l = n-1;
            while (k < l){
                int sum = num[i] + num[j] + num[k] + num[l];
                if (sum > target)
                    --l;
                else if (sum < target)
                    ++k;
                else {
                    vector<int> tmp;
                    tmp.push_back(num[i]);
                    tmp.push_back(num[j]);
                    tmp.push_back(num[k]);
                    tmp.push_back(num[l]);
                    ret.insert(tmp);
                    k++;
                    l--;
                }
            }
        }
    }
    vector<vector<int> > ret1;
    for (auto iter = ret.begin(); iter != ret.end(); ++iter) {
        ret1.push_back(*iter);
    }
    return ret1;
}
```

#### Letter Combinations of a Phone Number

- 题目链接：http://leetcode.com/onlinejudge#question_17
- 解题思路：
    - 当`input`为一位的时候，直接返回
    - 当`input`为多位的时候，将第一位取出，每个字符加上其它剩下的组合
- 实现：

```cpp
vector<string> letterCombinations(string digits) {
    string tables[] = {"abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    vector<string> ret;

    if (digits.empty()) {
        ret.push_back("");
        return ret;
    }
    if (digits.size() == 1){
        string str = tables[digits[0] - '2'];
        for (size_t i=0; i<str.size();i++){
            ret.push_back(str.substr(i,1));
        }
    } else {
        string str = tables[digits[0] - '2'];
        for (size_t i=0; i<str.size();i++){
            vector<string> last = letterCombinations(digits.substr(1, digits.size()-1));
            for (auto iter = last.begin(); iter != last.end(); ++iter) {
                string tmp = string(str.substr(i,1)) + *iter;
                ret.push_back(tmp);
            }
        }
    }

    return ret;
}
```

#### Remove Nth Node From End of List

- 题目链接：http://leetcode.com/onlinejudge#question_19
- 解题思路：链表的题目，题目本身很简单，只要注意边界情况就好了。
- 实现：

```cpp
ListNode *removeNthFromEnd(ListNode *head, int n) {
    int len = 1;
    for (ListNode *iter = head; iter->next != 0; iter = iter->next) {
        len++;
    }
    if (n > len) return 0;
    if (n== len) return head->next;
    ListNode *it = head;
    for (int i=0; i!=len-n-1; i++) {
        it = it->next;
    }
    if (it->next)
        it->next = it->next->next;
    else
        it->next = 0;
    return head;
}
```

#### Valid Parentheses

- 题目链接：http://leetcode.com/onlinejudge#question_20
- 解题思路：使用`stack`，左括号`push`，右括号时比较`front`与这个匹配不匹配。
然后在`pop`，最终栈中个数为0，则返回正确。
- 实现：

```cpp
bool isValidParentheses(string s){
    if (s.size() == 0 || s.size() % 2) return false;
    stack<char> temp;
    for (int i=0; i<s.size(); ++i) {
        if (s[i] == '{' || s[i] == '(' || s[i] == '[') {
            temp.push(s[i]);
            continue;
        }
        if (temp.size() == 0) return false;
        if (s[i] == ')' && temp.top() != '(')
            return false;
        if (s[i] == ']' && temp.top() != '[')
            return false;
        if (s[i] == '}' && temp.top() != '{')
            return false;
        temp.pop();
    }
    return (temp.size()) ? false : true;
}
```
