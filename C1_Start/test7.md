## 方向光与环境光

没有大自然的“[上帝算法](http://www.zhihu.com/question/28951564)”，计算机中光照的效果也是要计算的。对于目前的着色器，光照计算也分“逐顶点”和“逐片元”（或者说“逐像素”）。逐顶点计算光照和前面计算颜色、纹理相似，相邻顶点之间的像素通过插值计算得到，相当于把顶点之间理解成“平”的，这对于不平的面效果就会差些，于是会有逐片元的计算方法。

这一节我们对那个板条箱做逐顶点的光照，板条箱本来就是平的，逐顶点效果就挺好。

我们先了解计算机图形中的两种光：

1. 方向光（directional light）：从一个方向照过来的绝对平行的光。
2. 环境光（ambient light）：类似于真实世界中周围环境比如墙壁、空气灰尘散射之后，均匀地从各个方向照过来的光。

物体表面对光的两种反射：

1. 漫反射（Diffuse）：无论什么角度照射，都会朝所有方向反射；无论什么角度观看，亮度只取决于入射角（光线与入射表面法线的夹角）——入射角越大看上去越暗。
2. 镜面反射（Specular）：看到的物体亮度取决于视线是否沿着反射光线。通过调整镜面反射的程度模拟诸如木头、玻璃、金属等不同类型的表面。

Phong光照模型把这些类型综合在一起，所有的光具有两个特性：

1. 漫反射得到的RGB
2. 镜面反射得到的RGB

所有的材料具有四个特性：
1. 环境光反射的RGB
2. 漫反射得到的RGB
3. 镜面反射得到的RGB
4. 决定镜面反射细节的物体的光泽

场景中每个点的颜色，取决于自身颜色、灯光颜色、灯光效果。

根据Phong光照模型，要确完全确定场景中光照性质，需要每个光的两个属性和物体表面每个点的四个属性。

如图7是[维基百科上一个颜色相加得到整体颜色效果的例子](http://en.wikipedia.org/wiki/File:Phong_components_version_4.png)。

>![图7](../image/C1_Start/1_007.png)
>图7

我们将在shader里完成这些颜色相加的工作——计算每个点对环境光、漫反射、镜面反射的RGB的贡献，把这些RGB加起来。

要计算反射的效果，需要知道光线与模型表面的夹角，模型表面的方向是由法线向量来表达的。我们逐顶点计算光照，就要给出每个顶点的法线向量。

于是，我们要做的是：
* 为每个顶点保存一个法向量。
* 设定一个方向光的方向向量。
* 在顶点着色器中，对每个顶点计算表面方向与光的夹角并算出一个合适的RGB，再加上环境光的RGB。

```javascript
var cubeVertexNormalBuffer;
function initBuffers()
{
    //...
	cubeVertexNormalBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexNormalBuffer);
	var vertexNormals = [
					  // 正面
						0.0,  0.0,  1.0,
						0.0,  0.0,  1.0,
						0.0,  0.0,  1.0,
						0.0,  0.0,  1.0,

					  // 背面
						0.0,  0.0, -1.0,
						0.0,  0.0, -1.0,
						0.0,  0.0, -1.0,
						0.0,  0.0, -1.0,

					  // 顶部
						0.0,  1.0,  0.0,
						0.0,  1.0,  0.0,
						0.0,  1.0,  0.0,
						0.0,  1.0,  0.0,

					  // 底部
						0.0, -1.0,  0.0,
						0.0, -1.0,  0.0,
						0.0, -1.0,  0.0,
						0.0, -1.0,  0.0,

					  // 右侧面
						1.0,  0.0,  0.0,
						1.0,  0.0,  0.0,
						1.0,  0.0,  0.0,
						1.0,  0.0,  0.0,

					  // 左侧面
					  -1.0,  0.0,  0.0,
					  -1.0,  0.0,  0.0,
					  -1.0,  0.0,  0.0,
					  -1.0,  0.0,  0.0,
					];
	gl.bufferData(gl.ARRAY_BUFFER,
	    new Float32Array(vertexNormals), gl.STATIC_DRAW);
	cubeVertexNormalBuffer.itemSize = 3;
	cubeVertexNormalBuffer.numItems = 24;
    //...
}
```
很熟悉了，每个顶点再加一个属性——用来计算光的反射效果的方向向量。

```javascript
function drawScene()
{
    //...
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexNormalBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexNormalAttribute,
		cubeVertexNormalBuffer.itemSize, gl.FLOAT, false, 0, 0);
```
把顶点向量发给shader里对应的变量。
```javascript
    //..
	gl.activeTexture(gl.TEXTURE0);
	gl.bindTexture(gl.TEXTURE_2D, crateTexture);
	gl.uniform1i(shaderProgram.samplerUniform, 0);
```
纹理直接用MIPMAP那个方案，不再给三个切换。
```javascript

	var lighting = $("#lighting").is(":checked");
    gl.uniform1i(shaderProgram.useLightingUniform, lighting);
	if(lighting)
	{
		gl.uniform3f(
			shaderProgram.ambientColorUniform,
			parseFloat($("#ambientR").val()),
			parseFloat($("#ambientG").val()),
			parseFloat($("#ambientB").val())
			);
		var lightingDirection = [
			parseFloat($("#lightDirectionX").val()),
			parseFloat($("#lightDirectionY").val()),
			parseFloat($("#lightDirectionZ").val())
			];
		var adjustedLD = vec3.create();
		vec3.normalize(adjustedLD, lightingDirection);
		vec3.scale(adjustedLD, adjustedLD, -1);
		gl.uniform3fv(shaderProgram.lightingDirectionUniform, adjustedLD);
		gl.uniform3f(
			shaderProgram.directionalColorUniform,
			parseFloat($("#directionalR").val()),
			parseFloat($("#directionalG").val()),
			parseFloat($("#directionalB").val())
			);
	}
	//..
}
```
