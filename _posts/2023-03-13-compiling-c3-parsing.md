---
layout: post
title: 编译原理： Parsing
categories: Compiler
description: 编译原理第三章，Parsing
keywords: Compiler, 编译原理第三章，Parsing
---

此部分介绍编译原理课程第三章：**语法分析 (Parsing)** 和 **上下文无关文法 (Context-Free Grammar, CFG)** 的相关概念以及 **自顶向下分析 (Top-Down Parsing)** 的有关内容。

## 语法分析 (Parsing)
经过第二章Lexical Analysis的学习我们已经知道了可以用一些符号代表regular expression，例如：
```
digits = [0-9]+
sum = (digits "+")* digits
```
可以表示形如`28+301+9`这样的regular expression

有限自动机可以用来识别这样的regular expression，但有限自动机的局限性就是它的状态是有限的，对于另外一类问题，例如将上述expression改为形如`(28+301)`这样的带有平衡括号的表达式，有限自动机便无能为力，这是因为括号的嵌套深度可能超出有限自动机的状态数