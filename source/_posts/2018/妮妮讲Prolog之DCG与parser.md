---
title: 妮妮讲Prolog之DCG与parser
date: 2018-2-29
tags: prolog
---



每日一条Prolog小知识：应阿兰之邀，来讲讲dcg做简单的parser。

用dcg做parser，只需描述语法，然后就自动粗来一只吊炸天的parser啦。

今天用swi做例子，注意启动的时候要加上--triditional参数

dcg == definite clause grammar

swi prolog 7对字符串使用了新的类型和操作方法

所以要加上--triditional

加上这个参数后，就和sicstus一样啦

辣么我们今天就用一个例子来讲解

首先我们parse正整数，然后 parse 四则运算并计算

辣么，首先来parse单个数字

``` prolog
num1 ::= 0|1|2|3|4|5|6|7|8|9
```
意思是，一个数字，可以是0-9的每一个字

![图1](图片1.png)

对应的dcg代码要这么写

--> 是dcg专用符号

左边是谓词的头部

右边是对语法的描述

每个dcg谓词最后都多带两个参数

num1编译后是这个样子。

![图2](图片2.png)

辣么让我们来parse

![图3](图片3.png)

parse "345"这个字符串，"3"刚好是一个数字，剩余的在Rest

![图4](图片4.png)

开头空格不是数字，所以不符合语法，fail

![图5](图片5.png)

结尾是空，于是把语言的所有可能都整出来了。

字符串是list，所以"0"也就是[48]/

辣么我们来parse一个完整的整数

但是我们先不计算它的值

```prolog
simple_num ::= 
   num1
  | num1 simple_num
```
  
![图6](图片6.png)

没有少
|代表或者
a ::= b表示，a这个东西，可以被替换成b

辣么，假设simple_num对应345
num1是单个数字
num1 simple_num是指，num1后面接一个simple_num
空格是链接
意思是，345相当于num1(3) simple_num(45)，相当于num1(3) num1(4) simple_num(5)，相当于num1(3) num1(4) num1(5)。不想画图了，就这么简单的来讲吧。

这样吧，我做个parse_tree版本

![图7](图片7.png)

 这是生成parse tree的版本

也就是把刚才的那个奇怪的式子

变成树形结构

你看，就是简单的照着翻译哦

![图8](图片8.png)

效果是这样子的，如果不指定Rest，那么3, 34,345全都是整数，于是所有可能都列出来来

![图9](图片9.png)

辣么，简单的simple_num，照着翻译即可

![图10](图片10.png)

要注意的是，最后还是带哪两个参数，第一个是输入的字符串，第二个是消耗掉以后剩余的部分。

swi支持lazy list，这种方法可以直接用来parse网络数据流文件流等各种流

逼格网的爬虫也可以用这种方法一遍传输局，一遍解析

辣么我们来加点计算，让"345"变成数字345。

![11](图片11.png)

![12](图片12.png)

当然要一个初始值0，下边提供了

大括号是dcg专用符号

里面的代码不受dcg规则的影响

大括号外面的代码受dcg规则影响

无论如何都会带上两个参数

![13](图片13.png)

根据消耗的不同，得出不同的数字

这里用了十分简单的一个尾递归啦

![图14](图片14.png)

num/3, num/4（注意省略的两个参数实际上还是在的）
这个A用于保存num1的返回值
基本思路是，每parse到一个字符，就累计一下

![图15](图片15.png)

![图16](图片16.png)

不是，这叫unification

现在来说空格

空格简单的这样来定义吧。
ws ::= " " ws |  ""

![图17](图片17.png)

直接使用是这个样子

![图18](图片18.png)

意思是，要么是空白什么都没有（后边部分），要么是一个空格外加一个ws

辣么我们现在来计算四则运算表达式
首先我们这样定义四则运算表达式

![图19](图片19.png)

辣么，为了让他能适应更强的场景，比方说我们可以在数字和符号中间加空格，前后也可以加空格
辣么可以这样来

![20](图片20.png)

注意ws也包括空字符串

![21](图片21.png)

辣么，照着翻译就行啦

每一个都是第一行描述语法，第二行大括号做计算
实际使用是这样子的

![22](图片22.png)

注意最后一个
    34  + 4也是合法的表达式，所以也被尝试了

![23](图片23.png)


----------------------------------------------------------------

```prolog
num1(0) --> "0".
num1(1) --> "1".
num1(2) --> "2".
num1(3) --> "3".
num1(4) --> "4".
num1(5) --> "5".
num1(6) --> "6".
num1(7) --> "7".
num1(8) --> "8".
num1(9) --> "9".

num(Acc, X) -->
        num1(X1),
        { X is Acc * 10 + X1 }.
num(Acc, X) -->
        num1(A),
        { Acc1 is Acc * 10 + A },
        num(Acc1, X).
num(X) -->
        num(0, X).


simple_num -->
        num1(_).
simple_num -->
        num1(_),
        simple_num.



ws --> " ", ws.
ws --> "".



calc(X) -->
        ws, num(A), ws, "+", ws, num(B), ws,
        { X is A + B }.
calc(X) -->
        ws, num(A), ws, "-", ws, num(B), ws,
        { X is A - B }.
calc(X) -->
        ws, num(A), ws, "*", ws, num(B), ws,
        { X is A * B }.
calc(X) -->
        ws, num(A), ws, "/", ws, num(B), ws,
        { X is A / B }.


num1_parsetree(num1(0)) --> "0".
num1_parsetree(num1(1)) --> "1".
num1_parsetree(num1(2)) --> "2".
num1_parsetree(num1(3)) --> "3".
num1_parsetree(num1(4)) --> "4".
num1_parsetree(num1(5)) --> "5".
num1_parsetree(num1(6)) --> "6".
num1_parsetree(num1(7)) --> "7".
num1_parsetree(num1(8)) --> "8".
num1_parsetree(num1(9)) --> "9".

simple_num_parsetree(simple_num(X)) -->
        num1_parsetree(X).
simple_num_parsetree(simple_num(X, Y)) -->
        num1_parsetree(X),
        simple_num_parsetree(Y).
```
