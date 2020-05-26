---
tags: [C++]
title: Deciding which type to user in C++
created: '2020-03-26T10:25:02.785Z'
modified: '2020-03-26T10:49:56.837Z'
---

# Deciding which type to user in C++

- 当你很清楚数值不能为负的时候使用`unsigned`类型。
- 整型计算使用`int`型。实践中，`short`类型通常太小，`long`类型通常和`int`类型有一样的大小。如果数值超过了`int`类型的范围，就使用`long long`类型。
- 不要使用单纯的`char`和`bool`类型进行计算，而是用他们存储字符和真值。使用`char`进行计算往往是不安全的，因为在一些机器上，`char`是有符号的，在其他机器上则是无符号的。如果你需要一个很小的整型，请使用`unsigned char`或`signed char`。
- 使用`double`进行浮点型的计算。`float`通常没有足够的精度而且`double`类型的计算所产生的消耗也可忽略。事实上，在一些机器上，`double`类型的操作比`float`型更快。`long double`提供的精度通常是没有必要的，而且有相当大的时间消耗。
