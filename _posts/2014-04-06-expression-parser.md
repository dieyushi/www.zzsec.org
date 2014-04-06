---
layout: post
title: "表达式语法分析"
description: "expression parser"
category: coding
tags: [c, flex, bison]
published: true
comments: true
date: 2014-04-06 22:00:44 +0800
---
{% include JB/setup %}

### 前言

用过libpcap的程序员一定会注意到libpcap在获取数据包时允许设置过滤规则,例如`port 80`表示只获取HTTP流量。但这是怎么实现的呢？我们用简单的例子来说明。在例子之前，要先介绍待会要用到的`flex`和`bison`。

### flex/bison

flex和bison主要是为编译器和解释器的编程人员特别设计的工具，不过它们在其他领域也非常有用，因此也吸引了很多非编译器编程人员的注意。任何程序只要在输入中可以找到特定的模式，或者使用命令语言作为输入，都适合使用flex和bison。

<!--more-->

flex和bison是lex和yacc的GNU版本。flex用于词法分析，bison用于语法分析。简单来说，词法分析把输入分割成一个个有意义的词块，语法分析则确定这些词块是怎么关联的。举个C语言的例子:

    a = b + c;

词法分析器把代码分解为一个个的记号，a、等号、b、加号、c和分号;语法分析器则确定b+c是一个表达式，表达式的值被赋给了a。

### 一个与或表达式解析示例

了解了flex和bison的功能，我们来看一个示例。这个示例完成了字符串等于或者包含的与或运算。例如可以计算`ab == ab && ab ^^ a || ab == a`这样子的表达式。

    ./expr_parser "ab == ab && ab ^^ a || ab == a"
    Expression: (((((ab) == (ab))) && (((ab) ^^ (a)))) || (((ab) == (a))))
    Result: 1

关于词法，我们定义了与、或、等于、包含、括号、值这几种，对应着相应的运算。flex源文件的格式如下：

```
%{
#include <string.h>
#include "parser.h"
%}

%%
\n                              /* ignore end of line */
[ \t]+                          /* ignore whitespace */
"&&"                            return T_AND;
"||"                            return T_OR;
"=="                            return T_EQ;
"^^"                            return T_CT;
"("                             return '(';
")"                             return ')';
[a-zA-Z][a-zA-Z0-9]*            yylval.string_literal = strdup(yytext); return T_FIELD;
%%
```

语法分析把一系列的记号转化为语法分析树,bison的规则基本上就是BNF。bison源文件中BNF规则如下:

```
string_filter:
	T_FIELD T_EQ T_FIELD
		{
			$$ = make_op_node(EQ);
			$$->left_child = make_left_node($1);
			$$->right_child = make_right_node($3);
		}
	|
	T_FIELD T_CT T_FIELD
		{
			$$ = make_op_node(CT);
			$$->left_child = make_left_node($1);
			$$->right_child = make_right_node($3);
		}
	;
```

工程中定义了语法树的结构，并且在语法树生成后，遍历整个树并通过递归计算整个表达式的值。

整个工程可以参考https://github.com/dieyushi/expr_parser
