---
title: Markdown学习，一些常用小技巧
date: 2018-04-24 15:43:47
updated: 2018-04-24 15:43:47
categories: tech
tags: 
	- blog
	- 个人
description: 简单的学习一波Markdown，将自己平时写博客用的比较多的Markdown语法做做记录
toc: true
---
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

## 前言
因为本博客的创作都是使用的Markdown，把自己平时用的比较多的Markdown语法做一个简单的记录，方便以后的取用。

本文省略了一些基本的语法，例如标题、文本强调、无序有序列表、表格绘制等内容
## 详细技巧
### 图片位置大小
使用标准的图片插入方法`![]()`不能对图片进行位置和大小进行控制，默认居左并且按图片大小显示。

使用的方法是借助HTML标签来实现，在`<div>`标签中使用`align`、`width`、`height`等来控制，如下所示：

**示例代码**
```	
1. 默认显示效果
	![](../imgs/20180424_1.jpeg)
2. 位置大小控制效果
	<div align = "center"><img src = "https://i.loli.net/2018/04/24/5adf182dd449f.jpeg" width = "100"/></div>
```
**默认效果**
![](https://i.loli.net/2018/04/24/5adf182dd449f.jpeg)
**控制后的效果**
<div align = "center"><img src = "https://i.loli.net/2018/04/24/5adf182dd449f.jpeg" width = "100"/></div>

*PS*: 在文中插入图片除了可以将照片放在本地直接获取外，可以使用一些免费的CDN，例如[sm.ms](https://sm.ms/)，可以将图片传到线上，使用链接获取。

---
### 图文混排
图文混合编排也可以使用HTML中的标签来实现，如下所示的文字靠左，图片靠右的实现方式：

**示例代码**
```
<img src = "https://i.loli.net/2018/04/24/5adf182dd449f.jpeg" align = "right" width = "300">
```
**实例效果**

<div><img src = "https://i.loli.net/2018/04/24/5adf182dd449f.jpeg" align = "right" width = "300"></div>

描述1

描述2

描述3

描述4

这是一个表情表的介绍，为了凑字数多写一点，让在不同的屏幕大小的情况下都能看得出他的自动换行效果，紫薯布丁紫薯布丁紫薯布丁.....

---

### 段前缩进
在Markdown里，在一个空格或者TAB之后的其他缩进会默认被无视，因此需要使用`&ensp;` - 半角空格 或者 `&emsp;` - 全角空格来实现缩进
**示例代码**
```
这&ensp;中&ensp;间&ensp;有&ensp;半&ensp;角&ensp;空&ensp;格
&emsp;&emsp;这之前有全角空格
```
**实例效果**

这&ensp;中&ensp;间&ensp;有&ensp;半&ensp;角&ensp;空&ensp;格

&emsp;&emsp;这之前有全角空格

---

### 加强代码块
将需要高亮的代码块包裹在如下的格式内即可：
>```
>```语言名
>```


**Python效果**

``` python
def somefunc(a, b):
    return a + b

if __name__ == '__main__':
    print somefunc(2, 3)

```

---

### 文章目录
在hexo中的 `front-matter` 中填入 `toc: true` 即可

目录生成效果如本文所示

---

### 插入公式 MathJax方法
网上有很多教程的方法，大多数都是在hexo中安装上MathJax，这里介绍一种较为简洁的方法，在你文章的`front-matter`中插入一段代码：
```JavaScript
<script type="text/x-mathjax-config">
MathJax.Hub.Config({
tex2jax: {inlineMath: [['$','$'], ['\\(','\\)']]}
});
</script>
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>
```
之后，文中的公式两边加上$符号即可显示出公式，如下的`$E=mc^2$`和`$f(x) = x^2$`分别显示出了两个公式：

$E=mc^2$

$f(x) = x^2$

---

### 插入LaTeX公式

首先一个简单的示例，下述公式写法为：`$f'(x\_0)=\lim_{\Delta x\to 0} \frac{f(x\_0+\Delta x) - f(x\_0)}{\Delta x}$`

$f'(x\_0)=\lim_{\Delta x\to 0} \frac{f(x\_0+\Delta x) - f(x\_0)}{\Delta x}$

常见的总结如下：

|格式      	 |显示	      |
|:-----------:|:----------:|
|`\\(x^y\\)`|\\(x^y\\)|
|`\\(x^{y^z}\\)`|\\(x^{y^z}\\)|
|`\\(x_i\\)`|\\(x_i\\)|
|`\\(\alpha\\)`|\\(\alpha\\)|
|`\\(\beta\\)`|\\(\beta\\)|
|`\\(\Delta\\)`|\\(\Delta\\)|
|`$$\sum_{i=0}^n$$`|$$\sum_{i=0}^n$$|
|`$\frac{x}{y}$`|$\frac{x}{y}$|
|`\\(\sqrt 3\\)`|\\(\sqrt 3\\)|
|`\\(\sqrt[3] 5\\)`|\\(\sqrt[3] 5\\)|
|`$$\lim_{x \to 0}$$`|$$\lim_{x \to 0}$$|
|`$$f(x): \begin{cases} x, x>0 \\\ \\\ -x,x<0 \end{cases}$$`|$$f(x): \begin{cases} x, x>0 \\\ \\\ -x,x<0 \end{cases}$$|

---

## 参考：
* [http://daniellaah.github.io/2016/Mathmatical-Formula-within-Markdown.html](http://daniellaah.github.io/2016/Mathmatical-Formula-within-Markdown.html)
* [https://www.jianshu.com/p/0b257de21eb5](https://www.jianshu.com/p/0b257de21eb5)
* [https://hyxxsfwy.github.io/2016/01/15/Hexo-Markdown-%E7%AE%80%E6%98%8E%E8%AF%AD%E6%B3%95%E6%89%8B%E5%86%8C/](https://hyxxsfwy.github.io/2016/01/15/Hexo-Markdown-%E7%AE%80%E6%98%8E%E8%AF%AD%E6%B3%95%E6%89%8B%E5%86%8C/)
* [http://blog.mobing.net/content/hexo/hexo-mathjax.html](http://blog.mobing.net/content/hexo/hexo-mathjax.html)
