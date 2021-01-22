+++
title = "Markdown 语法简介"
description = "该文章展示基本的Markdown和HTML结合的语法格式"
author = "David Shu"
date = "2021-01-22T14:22:16+08:00"
tags = ["Markdown"]
categories = ["Markdown"]
comments = true
removeBlur = false
[[images]]
  src = "https://static.code-david.cn/blog/pic02.jpg"
  alt = "Desert Scene"
  stretch = "h"
+++

这篇文章主要介绍了一些基本的Markdown语法，补充介绍了结合HTML、CSS的技法
<!--more-->

## 多级大小的标题

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

## 引用

通过脚注和引用符号实现应用如下：

> 希望是本无所谓有，无所谓无的。这正如地上的路；其实地上本没有路，走的人多了，也便成了路。
> 
> <cite>鲁迅[^1]</cite>

[^1]: 上面的引用来自于鲁迅[《故乡》](https://baike.baidu.com/item/%E6%95%85%E4%B9%A1/2679148)

具体写法如下：
```
> 希望是本无所谓有，无所谓无的。这正如地上的路；其实地上本没有路，走的人多了，也便成了路。
> 
> <cite>鲁迅[^1]</cite>

[^1]: 上面的引用来自于鲁迅[《故乡》](https://baike.baidu.com/item/%E6%95%85%E4%B9%A1/2679148)
```

## Code Blocks
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

#### 有序列表

1. First item
2. Second item
3. Third item

实际写法：
```
1. First item
2. Second item
3. Third item
```

#### 无序列表

* List item
* Another item
* And another item

实际写法：
```
* List item
* Another item
* And another item
```

