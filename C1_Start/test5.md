## 引入纹理

三维模型表面有了自己的纹理，桌子才成为桌子，房子才成为房子。

纹理是用体现表面特征的图片贴上去的，就像装修贴墙纸一样。图片可以拉伸扭曲，假设我们用一张图贴一面墙，给出图左上贴墙左上，图右上贴墙右上……四个角都对应上，图就贴上去了。而位置的关系，用坐标来表示。

这一节我们学习如何贴纹理，为了减少些工作量，先把上一节代码所有四面体有关的部分删除，来给正方体贴纹理。

```javascript
function webGLStart()
{
    //...
	////initBuffers();
	initTexture();
	////gl.clearColor(0.0, 0.0, 0.0, 1.0);
	////gl.enable(gl.DEPTH_TEST);
	setTimeout("tick()", 100);
    //tick();
}
```
一开始加入纹理的初始化函数。tick()我们知道是执行动画的部分，直接用tick()的话，我看到了一个WARNING：

>[.WebGLRenderingContext]RENDER WARNING: texture bound to texture unit 0 is not renderable. It maybe non-power-of-2 and have incompatible texture filtering or is not 'texture complete'

我猜应该是JS异步执行这个特点造成的，纹理加载需要时间，在纹理还没加载完成的时候就开始绘制了，于是有了这个提示。用JS的setTimeout简单处理了一下，大意就是等“一小会儿”再绘制。也可以再加些标记什么的。这样WARNING就消失了。


```javascript
var myTexture;
function initTexture()
{
	myTexture = gl.createTexture();
	myTexture.image = new Image();
	myTexture.image.onload = function()
	{
		handleLoadedTexture(myTexture);
	}
	myTexture.image.src = "/Public/image/mytexture.jpg";
}
```
先用createTexture()建立一个纹理对象给myTexture，再让myTexture随身携带它的纹理的图片，是一个JS的image对象。为这个image对象的onload事件指定触发我们处理纹理的函数handleLoadedTexture()。

之后设置image的地址，可以是自己服务器的相对地址也可以是网络上图片的地址。指定地址后浏览器就开始在后台异步获取这个图片，获取成功后就触发了onload事件。
```javascript
function handleLoadedTexture(texture)
{
	gl.bindTexture(gl.TEXTURE_2D, texture);
```
gl.bindTexture和gl.bindBuffer相似，绑定texture为“当前纹理”，接下来没有新的绑定的话，对纹理的处理都是对texture的处理。
```javascript
	gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, 1);
```
gl.pixelstorei用于设置像素存储模式，第一个参数表示读入纹理的图片垂直翻转，这个是因为图片和纹理的坐标系统不同，翻过来就对应了，后面写纹理坐标会好写点。后一个参数表示存储器中每个像素行有1个字节对齐，我们先不用管。
```javascript
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,
	    gl.UNSIGNED_BYTE, texture.image);
```
gl.texImage2D，将刚读入的图片传入显卡纹理缓存，参数从前往后为：使用图片类型、细节层次（以后说）、显卡中储存格式（重复两次，以后说）、每个通道的规格（存储R、G、B等的数据类型）、图像对象本身。
```javascript
	gl.texParameteri(gl.TEXTURE_2D,
	    gl.TEXTURE_MAG_FILTER, gl.NEAREST);
	gl.texParameteri(gl.TEXTURE_2D,
	    gl.TEXTURE_MIN_FILTER, gl.NEAREST);
```
这两句分别表示目标区域比纹理图片大了或小了的时候如何处理，TEXTURE_MAG_FILTER和TEXTURE_MIN_FILTER分别是放大和缩小。NEAREST是使用纹理坐标中最接近的像素颜色作为需要绘制的颜色，速度快，效果看起来可能会有好多“块块”，不平滑。找个尺寸小点的图片当纹理，对比下gl.LINEAR试试看。
```javascript
	gl.bindTexture(gl.TEXTURE_2D, null);
}
```
最后把“当前纹理”绑定为空。这是不必要的，但是一种好习惯。

接着我们要在initBuffers()中设置纹理坐标。既然有纹理贴图，就不需要之前的颜色了，把设置颜色有关的变量和函数都替换为纹理相关的。

```javascript
var cubeVertexTextureCoordBuffer;
function initBuffers()
{
    //...
	cubeVertexTextureCoordBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexTextureCoordBuffer);
	textureCoords = [
					 // 正面
					 0.0, 0.0,
					 1.0, 0.0,
					 1.0, 1.0,
					 0.0, 1.0,

					 // 背面
					 1.0, 0.0,
					 1.0, 1.0,
					 0.0, 1.0,
					 0.0, 0.0,

					 // 顶部
					 0.0, 1.0,
					 0.0, 0.0,
					 1.0, 0.0,
					 1.0, 1.0,

					 // 底部
					 1.0, 1.0,
					 0.0, 1.0,
					 0.0, 0.0,
					 1.0, 0.0,

					 // 右侧面
					 1.0, 0.0,
					 1.0, 1.0,
					 0.0, 1.0,
					 0.0, 0.0,

					 // 左侧面
					 0.0, 0.0,
					 1.0, 0.0,
					 1.0, 1.0,
					 0.0, 1.0,
					];
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(textureCoords), gl.STATIC_DRAW);
	cubeVertexTextureCoordBuffer.itemSize = 2;
	cubeVertexTextureCoordBuffer.numItems = 24;
    //...
}
```
和之前设置点坐标、点颜色类似，纹理坐标也是点的一个属性（attribute），纹理坐标看做纹理图片上的一个比例，左下角是(0, 0)，右上角是(1, 1)，每个三维点对应纹理图片上按比例换算的一个位置，点与点之间也在这个比例上自动插值，显卡就知道了纹理图片与三维形状每个位置的对应关系。


