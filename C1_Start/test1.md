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
HTML文件中插入JS代码，需要先加入标签，JS代码写在标签下面。结束的时候当然对应也要加上：
```html
</script>
```
好，JS代码开始。
```javascript
$(document).ready(function ()
{
	webGLStart();
});

function webGLStart()
{
	var canvas = $("#test01-canvas")[0];
	initGL(canvas);
	initShaders();
	initBuffers();

	gl.clearColor(0.0, 0.0, 0.0, 1.0);
	gl.enable(gl.DEPTH_TEST);

	drawScene();
}
```
“$(document).ready(function (){});”是jQuery语法中，表示函数内的内容在Web页面加载完成时运行。如果不这样，可能在Web尚未加载完的时候，JS就开始尝试找某个控件，结果扑个空，代码逻辑就不能正常执行了。

webGLStart函数是一切的开始，首先我们用canvas的id“test01-canvas”找到它，$("#xxx")是jQuery用id找控件的方法，用class找的话就是$(".xxx")。找到之后得到的是jQuery对象，这里用一个数组下标[0]获取canvas的HTML对象，然后赋值给我们定义的canvas变量。

接下来三步是对不同部分的初始化，我们后面会说。

gl是我们自己定义的变量名，对应绘制的内容，后面会说它的初始化。

好比canvas是画布，gl是画笔，我们gl.clearColor四个参数，前三个RGB，第四个不透明度，这句就是让我们告诉gl：“如果没有新的通知，你就把整个画涂黑”。gl.enable用来开启一些功能，这里DEPTH_TEST是深度测试，现在我们先不管它有什么用，写到这就是。

drawScene()就是具体绘制操作的函数，也是我们自己定义的，后面会说。


###初始化“画笔”
```javascript
var gl;
function initGL(canvas)
{
	try
	{
		gl = canvas.getContext("webgl") || canvas.getContext("experimental-webgl");
		gl.viewportWidth = canvas.width;
		gl.viewportHeight = canvas.height;
	}catch(e){}
	if(!gl)
	{
		alert("无法初始化“WebGL”。");
	}
}
```

