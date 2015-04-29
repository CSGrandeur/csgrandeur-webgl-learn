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

一套代码流程的整体思路就是：

取得画布(canvas)->
从画布拿到画它的工具(gl)->
设置要画的内容（点的坐标，点之间的拓扑关系，存放到buffer里）->
告诉显卡要怎么画（从哪个方向看，点线面的颜色之类，用shader来交流）->
画！

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
这里我们定义了全局变量 gl，一般全局变量是不提倡的，第一节我们尽量少关注其他问题，这样方便一些。

canvas在入口里已经获取到了，canvas.getContext来拿到它的“内容”给gl画，之后gl无论画什么，都是在这个canvas的画布上。参数用了“experimental-webgl”和“webgl”并通过或运算连起来，因为webgl标准存在一个“实验阶段”，要通过“experimental-webgl”来获取内容，我现在直接只用“webgl”就可以，不过为了兼容性和其他不确定问题，两个都写上总保险些。

viewport顾名思义是视口，对于我们来说是在canvas上画，但是从另一个比较入戏的角度来说，计算机里放好了3D模型，我们是在选一个角度看，而且不一定是真的我们看，也可能是模仿一个蚂蚁去看，视野总是不同的，这里我们设置视口宽高和canvas画布大小一致。


###设置要画的内容

如果一个点一个点告诉显卡的话，累死我们，急死显卡。
把要画的内容所有的点存到一个缓冲区，对应显卡内存一块区域，让显卡一起处理，对数据规模较大又要即时演算做动画的情况，提高效率。
```javascript
var triangleVertexPositionBuffer;
var squareVertexPositionBuffer;
```

分别定义三角的buffer和矩形的buffer，刚定义的时候它们当然只是个什么都没有的变量。
```javascript
function initBuffers()
{
	triangleVertexPositionBuffer = gl.createBuffer
```
用gl给他们申请buffer。

```
	gl.bindBuffer(gl.ARRAY_BUFFER, triangleVertexPositionBuffer);
```
告诉WebGL，下面的代码在有新的变化之前，对gl.ARRAY_BUFFER操作就是对绑定的缓冲区（triangleVertexPositionBuffer）操作。
```javascript
	var vertices = [
	            	 0.0,  1.0,  0.0,
	            	-1.0, -1.0,  0.0,
	            	 1.0, -1.0,  0.0
	            	 ];
```
定义三角形的三个顶点，我们现在画平面，z坐标就设为0。
```javascript
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
```
前面已经绑定了ARRAY_BUFFER，这里把定义的三角形顶点坐标数组告诉它，也就是告诉了triangleVertexPositionBuffer。第三个参数STATIC_DRAW理解为“这数据的用途”，Float32Array和STATIC_DRAW这里先直接用，以后再解释。
```javascript
	triangleVertexPositionBuffer.itemSize = 3;
	triangleVertexPositionBuffer.numItems = 3;

```
itemSize和numItems并不是WebGL的内置变量，不过JavaScript这方面比较自用，变量不用定义就能用，让buffer自己携带相关信息方便许多。这里增加的信息就是明确“buffer里用vertices说明的九个数，表达的是三个顶点”。
```javascript
	squareVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, squareVertexPositionBuffer);
	vertices = [
	        	 1.0,  1.0,  0.0,
	        	-1.0,  1.0,  0.0,
	        	 1.0, -1.0,  0.0,
	        	-1.0, -1.0,  0.0
	        	];
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
	squareVertexPositionBuffer.itemSize = 3;
	squareVertexPositionBuffer.numItems = 4;

}
```
同理我们设置的矩形的数据。
