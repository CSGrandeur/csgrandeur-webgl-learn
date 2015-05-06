## 键盘交互与纹理过滤
这一节做一些简单的交互，并认识一下不同的纹理过滤类型。

效果如图6。

>![图6](../image/C1_Start/1_006.gif)

>图6

用四个方向键控制绕x轴和绕y轴的旋转，Page Up和Page Down键控制z轴方向上的放大与缩小，用F键在三种纹理过滤类型间切换。
```javascript
var xRot = 0;
var xSpeed = 0;
var yRot = 0;
var ySpeed = 0;
var z = -5.0;
var filter = 0;
```
与旋转、缩放、纹理过滤类型标记有关的全局变量。

```javascript
var crateTextures = Array();
function initTexture()
{
	var crateImage = new Image();
	for(var i = 0; i < 3; i ++)
	{
		var texture = gl.createTexture();
		texture.image = crateImage;
		crateTextures.push(texture);
	}
	crateImage.onload = function()
	{
		handleLoadedTexture(crateTextures);
	}
	crateImage.src = "/Public/image/crate.gif";
}
```
与上一节不同的是，我们创建了三个纹理对象，存到了一个数组里，纹理换成了一个板条箱的图片。这次传给handleLoadedTexture()的是一个数组。
```javascript
function handleLoadedTexture(textures)
{
	gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, 1);

	gl.bindTexture(gl.TEXTURE_2D, textures[0]);
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,
		gl.UNSIGNED_BYTE, textures[0].image);
	gl.texParameteri(gl.TEXTURE_2D,
		gl.TEXTURE_MAG_FILTER, gl.NEAREST);
	gl.texParameteri(gl.TEXTURE_2D,
		gl.TEXTURE_MIN_FILTER, gl.NEAREST);

	gl.bindTexture(gl.TEXTURE_2D, textures[1]);
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,
		gl.UNSIGNED_BYTE, textures[0].image);
	gl.texParameteri(gl.TEXTURE_2D,
		gl.TEXTURE_MAG_FILTER, gl.LINEAR);
	gl.texParameteri(gl.TEXTURE_2D,
		gl.TEXTURE_MIN_FILTER, gl.LINEAR);

	gl.bindTexture(gl.TEXTURE_2D, textures[2]);
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA,
		gl.UNSIGNED_BYTE, textures[0].image);
	gl.texParameteri(gl.TEXTURE_2D,
		gl.TEXTURE_MAG_FILTER, gl.LINEAR);
	gl.texParameteri(gl.TEXTURE_2D,
		gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_NEAREST);

	gl.generateMipmap(gl.TEXTURE_2D);

	gl.bindTexture(gl.TEXTURE_2D, null);
}
```
对传进来的纹理对象数组中三个对象使用了三种参数组合。
textures[0]和上一节一样，用的gl.NEAREST。
#### NEAREST
使用纹理坐标中最接近的一个像素，作为需要绘制的像素的颜色。边界分明，纹理图片太小的话会有马赛克的感觉。
#### LINEAR
使用纹理坐标中最接近的若干个颜色，通过加权平均得到需要绘制的像素的颜色。
#### MIPMAP
LINEAR方式在纹理放大到区域的时候比NEAREST效果好一些，但是在缩小的时候并没有明显差别，都会产生锯齿

可以在我们的交互页面上试试看，木板与木板之间有细缝，当缩小到大约1/10的时候，NEAREST和LINEAR模式都会在某些角度看上去一些细缝消失了。这个很好理解，当纹理需要缩小到一个尺寸的时候，需要绘制的两个模型上的点，换算到纹理上的位置，可能会刚好跨过细缝。

在纹理需要缩小比较多的情况下，如果想让LINEAR解决这个问题，就需要取更多的像素算均值，运算量会变大很多。MIPMAP方法是对原纹理生成了一个“图像金字塔”，这个是图像处理领域常用的结构，生成图像的1/2、1/4、1/8...，算一下等比数列求和会知道，至多只多占了一倍的空间，这在算法上是常数级的，可以接受。在纹理需要缩小时，选择最接近的纹理尺寸再做LINEAR处理。

于是有了这句gl.generateMipmap(gl.TEXTURE_2D)。





