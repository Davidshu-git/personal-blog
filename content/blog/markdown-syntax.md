+++
title = "Markdown 语法简介"
description = "该文章展示基本的Markdown和HTML结合的语法格式"
author = "David Shu"
date = "2021-01-22T14:22:16+08:00"
tags = ["Markdown"]
categories = ["Markdown"]
comments = true
removeBlur = true
[[images]]
  src = "https://static.code-david.cn/blog/markdown_pre1.png"
  alt = "Desert Scene"
  stretch = "v"
+++

Markdown是一种十分简便易上手的标记语言，十分适合个人开发者进行写作，这篇文章主要介绍了一些基本的Markdown语法
## 多级标题

Markdown支持六种大小的标题，下面展示了多达6级标题样式，在语法中，多级标题通过#开头表示，注意在#和文字之间添加空格，标题的级别随着#的个数增加而增加

# H1
## H2
### H3
#### H4
##### H5
###### H6

写法如下：
```
# H1
## H2
### H3
#### H4
##### H5
###### H6
```

## 段落

Markdown通过空行来形成段落

## 代码块
实现代码块的最简单方式是通过三个```符号将代码块部分进行包裹，这个符号就是键盘左上角在esc下方的键，英文输入法。此外四个空格的缩紧也可以实现代码块，示例如下：

    ```
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <title>Example HTML5 Document</title>
    </head>
    <body>
      <p>Test</p>
    </body>
    </html>
    ```

## 列表
列表实现，文字前跟*或数字+.需要注意的是中间有空格，这二者也可以结合嵌套使用

### 有序列表

1. First item
2. Second item
3. Third item

实际写法：
```
1. First item
2. Second item
3. Third item
```

### 无序列表

* List item
* Another item
* And another item

实际写法：
```
* List item
* Another item
* And another item
```

## 小结

Markdown作为最简单易上手的标记语言十分适合初学者，我们可以利用其进行写作而不必过多考虑版式问题，可让我们集中精力在写作之中，本篇文章简单介绍了其中很少部分但是使用率最高的语法，掌握了这几点就可以马上进行写作了，开始行动起来吧，如需要进一步的高级语法，如表格等，大家可以自行学习，全面介绍的文章在网络上随时可以找到