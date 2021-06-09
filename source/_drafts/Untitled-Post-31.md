---
title: Untitled Post - 31
tags: []
id: '292'
categories:
  - - uncategorized
---

MathJax.Hub.Config({"tex2jax": {"inlineMath": \[\['$','$'\], \['\\\\(','\\\\)'\]\]}}); MathJax.Hub.Config({"HTML-CSS": {"availableFonts":\["TeX"\],"scale": 150}});

/\*--------------------------------------------------------------------------------------------- \* Copyright (c) Microsoft Corporation. All rights reserved. \* Licensed under the MIT License. See License.txt in the project root for license information. \*--------------------------------------------------------------------------------------------\*/ body { font-family: "Segoe WPC", "Segoe UI", "SFUIText-Light", "HelveticaNeue-Light", sans-serif, "Droid Sans Fallback"; font-size: 14px; padding: 0 12px; line-height: 22px; word-wrap: break-word; } #code-csp-warning { position: fixed; top: 0; right: 0; color: white; margin: 16px; text-align: center; font-size: 12px; font-family: sans-serif; background-color:#444444; cursor: pointer; padding: 6px; box-shadow: 1px 1px 1px rgba(0,0,0,.25); } #code-csp-warning:hover { text-decoration: none; background-color:#007acc; box-shadow: 2px 2px 2px rgba(0,0,0,.25); } body.scrollBeyondLastLine { margin-bottom: calc(100vh - 22px); } body.showEditorSelection .code-line { position: relative; } body.showEditorSelection .code-active-line:before, body.showEditorSelection .code-line:hover:before { content: ""; display: block; position: absolute; top: 0; left: -12px; height: 100%; } body.showEditorSelection li.code-active-line:before, body.showEditorSelection li.code-line:hover:before { left: -30px; } .vscode-light.showEditorSelection .code-active-line:before { border-left: 3px solid rgba(0, 0, 0, 0.15); } .vscode-light.showEditorSelection .code-line:hover:before { border-left: 3px solid rgba(0, 0, 0, 0.40); } .vscode-dark.showEditorSelection .code-active-line:before { border-left: 3px solid rgba(255, 255, 255, 0.4); } .vscode-dark.showEditorSelection .code-line:hover:before { border-left: 3px solid rgba(255, 255, 255, 0.60); } .vscode-high-contrast.showEditorSelection .code-active-line:before { border-left: 3px solid rgba(255, 160, 0, 0.7); } .vscode-high-contrast.showEditorSelection .code-line:hover:before { border-left: 3px solid rgba(255, 160, 0, 1); } img { max-width: 100%; max-height: 100%; } a { color: #4080D0; text-decoration: none; } a:focus, input:focus, select:focus, textarea:focus { outline: 1px solid -webkit-focus-ring-color; outline-offset: -1px; } hr { border: 0; height: 2px; border-bottom: 2px solid; } h1 { padding-bottom: 0.3em; line-height: 1.2; border-bottom-width: 1px; border-bottom-style: solid; } h1, h2, h3 { font-weight: normal; } h1 code, h2 code, h3 code, h4 code, h5 code, h6 code { font-size: inherit; line-height: auto; } a:hover { color: #4080D0; text-decoration: underline; } table { border-collapse: collapse; } table > thead > tr > th { text-align: left; border-bottom: 1px solid; } table > thead > tr > th, table > thead > tr > td, table > tbody > tr > th, table > tbody > tr > td { padding: 5px 10px; } table > tbody > tr + tr > td { border-top: 1px solid; } blockquote { margin: 0 7px 0 5px; padding: 0 16px 0 10px; border-left: 5px solid; } code { font-family: Menlo, Monaco, Consolas, "Droid Sans Mono", "Courier New", monospace, "Droid Sans Fallback"; font-size: 14px; line-height: 19px; } body.wordWrap pre { white-space: pre-wrap; } .mac code { font-size: 12px; line-height: 18px; } pre:not(.hljs), pre.hljs code > div { padding: 16px; border-radius: 3px; overflow: auto; } /\*\* Theming \*/ .vscode-light, .vscode-light pre code { color: rgb(30, 30, 30); } .vscode-dark, .vscode-dark pre code { color: #DDD; } .vscode-high-contrast, .vscode-high-contrast pre code { color: white; } .vscode-light code { color: #A31515; } .vscode-dark code { color: #D7BA7D; } .vscode-light pre:not(.hljs), .vscode-light code > div { background-color: rgba(220, 220, 220, 0.4); } .vscode-dark pre:not(.hljs), .vscode-dark code > div { background-color: rgba(10, 10, 10, 0.4); } .vscode-high-contrast pre:not(.hljs), .vscode-high-contrast code > div { background-color: rgb(0, 0, 0); } .vscode-high-contrast h1 { border-color: rgb(0, 0, 0); } .vscode-light table > thead > tr > th { border-color: rgba(0, 0, 0, 0.69); } .vscode-dark table > thead > tr > th { border-color: rgba(255, 255, 255, 0.69); } .vscode-light h1, .vscode-light hr, .vscode-light table > tbody > tr + tr > td { border-color: rgba(0, 0, 0, 0.18); } .vscode-dark h1, .vscode-dark hr, .vscode-dark table > tbody > tr + tr > td { border-color: rgba(255, 255, 255, 0.18); } .vscode-light blockquote, .vscode-dark blockquote { background: rgba(127, 127, 127, 0.1); border-color: rgba(0, 122, 204, 0.5); } .vscode-high-contrast blockquote { background: transparent; border-color: #fff; } /\* Tomorrow Theme \*/ /\* http://jmblog.github.com/color-themes-for-google-code-highlightjs \*/ /\* Original theme - https://github.com/chriskempson/tomorrow-theme \*/ /\* Tomorrow Comment \*/ .hljs-comment, .hljs-quote { color: #8e908c; } /\* Tomorrow Red \*/ .hljs-variable, .hljs-template-variable, .hljs-tag, .hljs-name, .hljs-selector-id, .hljs-selector-class, .hljs-regexp, .hljs-deletion { color: #c82829; } /\* Tomorrow Orange \*/ .hljs-number, .hljs-built\_in, .hljs-builtin-name, .hljs-literal, .hljs-type, .hljs-params, .hljs-meta, .hljs-link { color: #f5871f; } /\* Tomorrow Yellow \*/ .hljs-attribute { color: #eab700; } /\* Tomorrow Green \*/ .hljs-string, .hljs-symbol, .hljs-bullet, .hljs-addition { color: #718c00; } /\* Tomorrow Blue \*/ .hljs-title, .hljs-section { color: #4271ae; } /\* Tomorrow Purple \*/ .hljs-keyword, .hljs-selector-tag { color: #8959a8; } .hljs { display: block; overflow-x: auto; color: #4d4d4c; padding: 0.5em; } .hljs-emphasis { font-style: italic; } .hljs-strong { font-weight: bold; } /\* \* Markdown PDF CSS \*/ pre { background-color: #f8f8f8; border: 1px solid #cccccc; border-radius: 3px; overflow-x: auto; white-space: pre-wrap; overflow-wrap: break-word; } pre:not(.hljs) { padding: 23px; line-height: 19px; } blockquote { background: rgba(127, 127, 127, 0.1); border-color: rgba(0, 122, 204, 0.5); } .emoji { height: 1.4em; } /\* for inline code \*/ :not(pre):not(.hljs) > code { color: #C9AE75; /\* Change the old color so it seems less like an error \*/ font-size: inherit; }

# 算法复杂性分析

## 基础术语

（1）数据（Data）是能被计算机处理的符号或符号集合，含义广泛，可理解为“原材料”。如字符、图片、音视频等。

（2）数据元素（data element）是数据的基本单位。例如一张学生统计表。

（3）数据项（data item）组成数据元素的最小单位。例如一张学生统计表，有编号、姓名、性别、籍贯等数据项。

（4）数据对象（data object）是性质相同的数据元素的集合，是数据的一个子集。例如正整数N={1，2，3，····}。

（5）数据结构（data structure）是数据的组织形式，数据元素之间存在的一种或多种特定关系的数据元素集合。

（6）数据类型（data type）是按照数据值的不同进行划分的可操作性。在C语言中还可以分为原子类型和结构类型。原字类型是不可以再分解的基本类型，包括整型、实型、字符型等。结构类型是由若干个类型组合而成，是可以再分解的。

### 1.2数据的逻辑结构

逻辑结构（logical structure）是指在数据中数据元素之间的相互关系。数据元素之间存在不同的逻辑关系构成了以下4种结构类型。

（1）集合结构：集合的数据元素没有其他关系，仅仅是因为他们挤在一个被称作“集合”的盒子里。

（2）线性结构：线性的数据元素结构关系是一对一的，并且是一种先后的次序，就像a-b-c-d-e-f-g·····被一根线穿连起来。

（3）树形结构：树形的数据元素结构关系是一对多的，这就像公司的部门级别，董事长-CEO\\CTO-技术部\\人事部\\市场部.....。

（4）图结构：图的数据元素结构关系是多对多的。就是我们常见的各大城市的铁路图，一个城市有很多线路连接不同城市。  
（轉載自：https://blog.csdn.net/csdn\_aiyang/article/details/84837553 ）

## 大O表示法运算法则

*   加法规则： f1(n) + f2(n) = O(max( f1(n) , f2(n))
    *   顺序结构，if结构，switch结构
*   乘法规则： f1(n) \* f2(n) = O(max( f1(n) \* f2(n))
    *   for ， while ，do-while结构

### 算法时间复杂度分析

语句频度 = $\\sum\_{i=1}^n\\sum\_{j=1}^i\\sum\_{k=1}^j1=\\sum\_{i=1}^n\\frac{i(i+1)}{2}$

```
for(i = 1;i < = n;i++) 
  for(j = 1;j< = i; = i;j++) 
   for(k = 1;k <= j;k++) 
     x=x+1;
```

语句频度 =  
$$\\sum\_{i=1}^n\\sum\_{j=1}^i\\sum\_{k=1}^j1=\\sum\_{i=1}^n\\frac{i(i+1)}{2}$$ $$= \\frac{1}{2}(\\sum\_{i=1}^ni^2 + \\sum\_{i=1}^ni)$$