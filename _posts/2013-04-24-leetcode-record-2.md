---
layout: post
title: "leetcode刷题记录2"
description: "leetcode coding record"
category: coding
tags: [c++, algorithm, leetcode]
published: true
disqus: y
date: 2013-04-24 11:35:46 +0800
---


昨天晚上有别的事情，没来的记录和总结，今天上午补上，大概做了六七个小时吧，解决了7道题，难度都是中低等的题目。还是要继续提高啊。

### 题目

#### Reverse Integer

- 题目链接：[http://leetcode.com/onlinejudge#question_7](http://leetcode.com/onlinejudge#question_7)
- 解题思路：很简单的一个题目，从最高位开始，得到每一位的值，组起来就可以了。
- 实现：

```cpp
int reverse(int x){
    int ret = 0;

    while (x){
        ret = ret * 10 + x % 10;
        x = x / 10;
    }

    return ret;
}
```

<!--more-->

#### String to Integer (atoi)

- 题目链接：[http://leetcode.com/onlinejudge#question_8](http://leetcode.com/onlinejudge#question_8)
- 解题思路：这道题最麻烦的地方在于`+-`符号以及`int`值范围的考虑，自己实现了，但是很ugly，各位童鞋可以考虑参考下`glibc`里面的实现方法(代码位于`stdlib/atoi.c`，只是封装了`strtol`，位于`stdlib/strtol.c`)。
- 实现：

```cpp
int StringToInterger(char *str) {
    int len = strlen(str);
    long long ret = 0;
    bool num_existed = false;
    int sign = 1;
    bool exist_signed = false;
    for (int i=0; i<len; i++){
        if (str[i] == ' ') {
            if (num_existed) {
                break;
            }
            continue;
        }
        if (str[i] == '+' && !num_existed) {
            if (exist_signed)
                return 0;
            int temp = i+1;
            exist_signed = true;
            if ((temp <= len) && (str[temp] != ' '))
                continue;
            else
                return 0;
        }
        if (str[i] == '-' && !num_existed) {
            if (exist_signed)
                return 0;
            sign = -1;
            int temp = i+1;
            exist_signed = true;
            if ((temp <= len) && (str[temp] != ' '))
                continue;
            else
                return 0;
        }
        if (str[i] > '9' || str[i] < '0') {
            if (num_existed)
                break;
            return 0;
        }
        int tmp = str[i] - '0';
        ret = ret * 10 + tmp;
        num_existed = true;
    }
    ret = sign * ret;
    if (ret > INT_MAX)
        return INT_MAX;
    if (ret < INT_MIN)
        return INT_MIN;
    return ret;
}
```

#### Palindrome Number

- 题目链接：[http://leetcode.com/onlinejudge#question_9](http://leetcode.com/onlinejudge#question_9)
- 解题思路：题目中说明了不要用上面那种反转字符串再比较的方法。要求解题不使用额外的空间。依次取第`i`位和`length-i-1`为比较。注意负数不是Palindrome Number。
- 实现：

```cpp
bool isPalindrome(int x) {
    if (x < 0)
        return false;

    int len = 0;
    int temp = x;
    while (temp) {
        len++ ;
        temp = temp / 10;
    }
    for (int i=0; i < len/2; i++){
        int head, tail = 0;
        head = x / std::pow(10, len-i-1);
        if (i) {
            head = head % 10;
        }

        tail = x % int(std::pow(10, i+1));

        int j = i;
        while (j) {
            tail = tail / 10;
            j--;
        }

        if (tail == head){
            continue;
        }
        return false;
    }
    return true;
}
```

#### Container With Most Water

- 题目链接：[http://leetcode.com/onlinejudge#question_11](http://leetcode.com/onlinejudge#question_11)
- 解题思路：一开始居然没有看懂题目，弄了半天才搞清楚是什么意思。稍微解释下，参数是`vector<int> &height`, 其中的第i位与第i位的值`(i, height[i])`是坐标平面上的一个点，它与x轴做垂直线，这样就形成了i个垂直于x轴的线，假设这些是盛水的段短，求任意两个短板间水的容量最大值。一般人很容易就直接写两个循环求解了，就像这样：

```cpp
int maxArea(vector<int> &height){
    int maxcap = 0;
    size_t left = 0, right = height.size()-1;

    for (auto ileft = height.begin(); ileft < height.end(); ++ileft) {
        for (auto irighr = ileft; irighr < height.end(); ++irighr) {
            int cap = min(*ileft, *irighr) * (irighr - ileft);
            maxcap = max(maxcap, cap);
        }
    }

    return maxcap;
}
```

跑小集合没问题，但是这个是通不过大集合的，时间复杂度太高了。最终想到一个解决方法：取最左边的为`left`，最右边的为`right`，所有可能比这种大的情况只能是这两个高度较小的那一端向中间移动。一直循环这个动作，最大值肯定在其中。

- 实现：

```cpp
int maxArea(vector<int> &height) {
    int maxcap = 0;
    size_t left = 0, right = height.size() - 1;

    while (left < right) {
        int cap = (right - left) * min(height[left], height[right]);
        maxcap = max(cap, maxcap);

        if (height[left] < height[right])
            ++left;
        else
            --right;
    }
    return maxcap;
}
```

#### Integer to Roman

- 题目链接：[http://leetcode.com/onlinejudge#question_12](http://leetcode.com/onlinejudge#question_12)
- 解题思路：根据罗马字母的规矩，逐次减相应的值(1000,900,500,400,100,90,50,40,10,9,5,4,1)，加上相应的罗马字符。
- 实现：

```cpp
string IntToRoman(int num) {
    int i = 0;
    string ret;
    int a[]={1000,900,500,400,100,90,50,40,10,9,5,4,1};
    string r[]={"M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I"};

    while (num > 0) {
        while (num >= a[i]) {
            ret = ret + r[i];
            num = num - a[i];
        }
        i++;
    }
    return ret;
}
```

#### Roman to Integer

- 题目链接：[http://leetcode.com/onlinejudge#question_13](http://leetcode.com/onlinejudge#question_13)
- 解题思路：遍历每一个字符，如果其对应的值比前一个的值小，直接加上这个值，反之则要先减去2倍的前一位的值，再加上这个的值。自己都给我自己绕晕了，直接看代码吧。
- 实现：

```cpp
int romanToInt(string s) {
    if (s.empty())
        return 0;

    map<char, int> rt;
    rt['I']=1;
    rt['V']=5;
    rt['X']=10;
    rt['L']=50;
    rt['C']=100;
    rt['D']=500;
    rt['M']=1000;

    auto iter = s.begin();
    int ret = rt[*iter];
    int pre = rt[*iter];
    iter++;
    for (; iter != s.end(); ++iter) {
        if(rt[*iter] <= pre)
            ret += rt[*iter];
        else
            ret = ret - 2 * pre + rt[*iter];
        pre = rt[*iter];
    }
    return ret;
}
```

#### Longest Common Prefix

- 题目链接：[http://leetcode.com/onlinejudge#question_14](http://leetcode.com/onlinejudge#question_14)
- 解题思路：首先确定所有字符串的最短值，从第一个字符串的第一位依次往后取，每次都在其它字符串中判断起始的这几位是否相同，不同就直接返回，一直到最短长度。
- 实现：

```cpp
string longestCommonPrefix(vector<string> &strs) {
    if (strs.size() == 0){
        return "";
    }

    int minlen = strs.begin()->length();
    for (auto iter = strs.begin(); iter != strs.end(); ++iter) {
        minlen = min(minlen, int(iter->length()));
    }
    string prefix;
    int i = 1;
    while (i <= minlen) {
        prefix =  strs.begin()->substr(0,i);
        for(auto iter = strs.begin(); iter != strs.end(); ++iter){
            if (iter->substr(0, prefix.length()) != prefix) {
               return prefix.substr(0, prefix.length()-1);
            }
        }
        i++;
    }
    return prefix;
}
```
