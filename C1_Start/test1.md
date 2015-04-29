##三角与矩形
首先记得在自己的代码最顶部**加上本章开头说的jQuery与gl-matrix**的引入代码，后面我们的示例代码中不再说明。

本节我们的目标是在Canvas里画一个如图1的三角和方块。

![图1](../image/C1_Start/1_1_001.png)

### 放控件
首先把canvas放到网页上
```html
<canvas id = "test01-canvas" width = "800" height = "600"></canvas>
```
canvas可以理解为一个画布，它不止可以做WebGL，还可以做画图等其他事情。

画布的大小需要在这里设置，或用JS修改，如果用css文件去设置尺寸的话，会发现最后绘制出来的东西被扭曲了，因为css改变了控件的大小但没有改变画布大小。

id是控件的唯一标识，原则上你不能写两个canvas用同一个id。我们会在JS代码中通过这个id找到这个canvas。
### 程序的入口
接下来就开始我们的JS代码了。再提一次我们前面说过引入jQuery，下面的代码虽然用到jQuery地方很少，但是不引入jQuery的话代码是跑不动的。
```html
<script type="text/javascript">
```
