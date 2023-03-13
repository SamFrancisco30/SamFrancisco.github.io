---
layout: post
title: 编译原理： Parsing
categories: Compiler
description: 编译原理第三章，Parsing
keywords: Compiler, 编译原理第三章，Parsing
---

此部分介绍 **语法分析 (Parsing)** 和 **上下文无关文法 (Context-Free Grammar, CFG)** 的相关概念以及 **自顶向下分析 (Top-Down Parsing)** 的有关内容。

## 语法分析 (Parsing)
经过第二章Lexical Analysis的学习我们已经知道了可以用一些符号代表regular expression，例如：
```
digits = [0-9]+
sum = (digits "+")* digits
```
可以表示形如`28+301+9`这样的regular expression