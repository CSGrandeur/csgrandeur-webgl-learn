# 第一章 起步
本章从一个能够运行的WebGL基础代码开始

我们不考虑HTML的结构，比如header了body了什么的，只关注WebGL相关代码，而且这样的代码是可以用的~~

当然为了更好的体验，可以网上找个HTML页面代码，再加上些css文件弄美观些，然后把WebGL代码写在<span style="color:#0000ff;">&lt;body&gt;&lt;/body&gt;</span>里。
## **准备工作**


### 浏览器

比较新的浏览器基本没什么问题，一般决定学习WebGL的同学多少有点极客精神，不至于用太老旧的浏览器吧。

目前来说Chrome一般是开发者的首选，如果你用的不是Chrome浏览器，随便上网搜一些WebGL的例子打开看自己的浏览器能不能正常显示应该也差不多。

如果我什么时候弄了个服务器，就把每一节的练习源码放上去，到时候会把链接放在这里，到时也可以用这些页面测试浏览器是否支持。

所以，先准备一个最新的浏览器吧。

### 开发环境
Web一般是用浏览器来看，所以用不到什么编译器，那么像UltraEdit、Sublime，或者不怎么极客的我没用过的Vim和Emacs口碑这么好应该也可以。

之前做项目已经习惯了Eclipse，可以装各种扩展，语法高亮、代码提示什么的都不错，就是配置到很顺手还是要花点功夫的。

建议做个本地服务器，毕竟WebGL是在Web上的，有个服务器调试代码能有更好的Web体验。我就用[XAMPP](www.apachefriends.org)了，装上之后根目录的htdocs里放自己的Web页面，就可以在浏览器用127.0.0.1访问了。当然也可以修改它的apache配置文件满足自己的其他需要，不再赘述。


### 如何调试

前端程序员应该很熟悉，大多浏览器用F12可打开调试器，在Console里查看页面错误。

可以利用调试器点选网页元素，看元素的代码位置、内容和相关的css，帮助解决一些基本的前端问题。
太高深的我也不懂，就不多说了。了解一些前端知识总是有好处的。


### 使用的外部库

#### jQuery

jQuery是一个家喻户晓的JS库，大家一般不太喜欢直面JS原生代码，使用jQuery会方便很多。大家都在用，不用担心冷门什么的，而且也不会影响对WebGL知识的学习。

jQuery官网：http://jquery.com/

jQuery开源主页：https://github.com/jquery/jquery

可以在自己WebGL代码页面的最顶端加上一行
```html
<script src="jquery.min.js" type="text/javascript"></script>
```
当然src=要加上能找到jquery.min.js文件的相对路径。.js前面有个min的 意思是被压缩过的js，功能和不加min的一样，但文件更小加快网页打开速度。如果需要调试时候查看源码，就用不带min的版本。

也可以用CDN：
```html
<script src="http://code.jquery.com/jquery-2.1.4.min.js" type="text/javascript"></script>
```
用CDN的前提是查看网页的这台电脑要联网。
#### gl-matrix
做GL会涉及很多图形变换，在计算机中对坐标的变换是通过矩阵来实现的，比如可以去了解一下仿射变换。WebGL并没有集成矩阵的运算，我们专注WebGL的学习，没有必要一开始花费太多精力去写矩阵操作，轮子造好了，请享用。

gl-matrix开源主页：https://github.com/toji/gl-matrix

仓库的dist文件夹下有gl-matrix-min.js和gl-matrix.js两个文件。同样在页面顶端加入
```html
<script src="gl-matrix-min.js" type="text/javascript"></script>
```

好，我们开始吧 ![](../image/general/lovelysmile.png)
