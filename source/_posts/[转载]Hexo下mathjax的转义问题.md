---
title: 【转载】Hexo下mathjax的转义问题
date: 2017-05-17 11:02:35
categories: 经验心得
mathjax: true
tags: 经验心得
description: 本文转载自博客 天空的城。markdwon和mathjax在语法上存在转义的冲突。我因为该问题折腾了半天，本文提供了该问题的最终解决方案，感谢作者。
---
&emsp;&emsp;本文转载自博客[天空的城](http://shomy.top/2016/10/22/hexo-markdown-mathjax/)。markdwon和mathjax在语法上存在转义的冲突。我因为该问题折腾了半天，本文提供了该问题的最终解决方案，感谢作者。
## 问题
&emsp;&emsp;我们平时使用markdown写文档的时候，免不了会碰到数学公式，好在有强大的Mathjax,可以解析网页上的数学公式，与hexo的结合也很简单，可以手动加入js，或者直接使用hexo-math插件.大部分情况下都是可以的，但是Markdwon本身的特殊符号与Latex中的符号会出现冲突的时候:  
—的转义，在markdown中，`_`是斜体，但是在latex中，却有下标的意思，就会出现问题。  
`\\`的换行，在markdown中，`\\`会被转义为`\`这样也会影响影响mathjax对公式中的`\\`进行渲染  
## 原因
&emsp;&emsp;hexo默认使用marked.js去解析我们写的markdown，比如一些符号，`_`代表斜体，会被处理为`<em>`标签，比如`x_i`在开始被渲染的时候，处理为`x<em>i</em>`，这个时候mathjax就无法渲染成下标了。很多符号都有这个问题，比如粗体`*`,也是无法在mathjax渲染出来的，好在有替代的乘法等,包括`\\`同理。 所以说到底，是hexo使用的markdown引擎的锅，因为很多其它引擎在这方面处理的很好。

## 解决方法

在网上查了一些资料，结合自己使用经验，总结为如下的方法

### 方法一：手动escape  
&emsp;&emsp;这个方法最直接，需要转义我就转义。比如我需要在公式中写下标符号，那就修改写法写为: $x\_i$;需要换行就使用`\\`,即可。 很明显，这种方式虽然可以解决问题通用性很差,比如想迁移到其它地方，就无法识别了，因为大部分的markdown引擎是没有这个问题的。

### 方法二：保护代码块  
&emsp;&emsp;我们知道在markdown中的两个`` ` ``之间的东西不会转义，正好符号我们的需求，我们保护``\[``之间的代码不被markdown渲染，这样就可以，但是有一个后遗症，就是的样式问题，markdown中是code的样式的，这样也还是有问题，解决思路就是重新处理code标签,使其遇见`\]`的话，跳过，详情见:[解决MathJax与Markdown的冲突](https://liam0205.me/2015/09/09/fix-conflict-between-mathjax-and-markdown/)这个方法有个问题，就是有时候我们的代码中会有$$的出现，这时候，仍然使用mathjax就有可能出现无法预料的结果了。

### 方法三：更换Hexo的markdown引擎  
&emsp;&emsp;这个是最重的方法，直接换发动机，就是把hexo默认的渲染markdown的引擎换掉。查到了有如下几个插件可以使用:

*hexo-renderer-pandoc*，很强大的解析器，Pandoc的语法完全没有上述问题, 推荐使用。
*hexo-renderer-kramed*,或者*hexo-renderer-marked*，这两个差不多，第一个fork了第二个，改了一部分bug，但是还是有部分问题。  

&emsp;&emsp;下面说一下如何使用pandoc渲染。  

1. 首先安装Pandoc，官网提供了Ubuntu的deb安装包，按照官网教程就可以安装完成。  
2. 就是卸载hexo默认的markd,再安装新的:  

```shell
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-pandoc --save
```
  
如果决定要换的话，还是更换为Pandoc吧，虽然比较笨重，需要首先安装Pandoc， 不过的确可以完美解决上述的不兼容问题，另外它的语法与markdown有些微的差异，需要注意。  

### 方法四：修改Hexo渲染源码  
这个方法是我目前使用的，相对来说，通用性较高的一种方式。思路就是修改hexo的渲染源码: `nodes_modules/marked/lib/marked.js`:  

 - 去掉`\\`的额外转义  
 - 将em标签对应的符号中，去掉`_`，因为markdown中有`*`可以表示斜体，`—`就去掉了。  

具体思路参考了[使Marked.js与MathJax共存](http://blog.csdn.net/emptyset110/article/details/50123231)：  

打开`nodes_modules/marked/lib/marked.js`:   

第一步: 找到下面的代码:  

```javascript
escape: /^\\([\\`*{}\[\]()# +\-.!_>])/,
```  

改为:  

```javascript
escape: /^\\([`*{}\[\]()# +\-.!_>])/,
```  

这样就会去掉`\\`的转义了。   
第二步: 找到em的符号:  

```javascript
em: /^\b_((?:[^_]|__)+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```  

改为:   

```javascript
em:/^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```  

去掉`_`的斜体含义，这样就解决了。这种方式指标不治本，因为保证不了还可能有其它的字符会冲突，这样的话，还需要返回去接着修改。但是通用性很高，因为我们没有修改文章的内容，可以放到别的引擎下也基本上顺利渲染。
建议使用最后两种方法。
