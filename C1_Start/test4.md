## 来点真正的3D

这节我们把三角改成四面体，矩形改成正方体。

效果如图4。


>![图4](../image/C1_Start/1_1_004.gif)

>图4

之前的平面图形是一个面，立体的图形是由多个面组成的，我们一个面一个面地给出顶点坐标，画上去就可以了。

在绘制正方体的时候，我们增加一个索引。原因是：
1. 一方面相邻的面是有公共点的，如果所有面都各自用坐标表示，就多了许多重复工作，所以这时我们增加一个索引（index），这样只需给出所有点的坐标，然后用索引来重复使用点。
2. 另一方面如果我们用多边形画面的时候，接口不清楚我们每个面有几条边，就要用for循环去一个面一个面画。如果用“TRIANGLE_STRIP”来画，则这个面画完，可能会和另一个面的顶点组成一个我们并不想画的三角形。如果用“TRIANGLES”来画，一个面四个点，等于两个三角形，两个公共点的坐标我们就需要多提供一次。使用索引+“TRIANGLES”，就方便多了。


为了让变量名更有意义一些，先把所有的triangle改成pyramid，square改成cube。比如rTri改为rPyramid，triangleVertexPositionBuffer改成pyramidVertexPositionBuffer等等。

我们这里四面体对所有面都分别给出点坐标，正方体用点索引的方法。

```javascript
var cubeVertexIndexBuffer;
```
增加全局变量，索引的buffer。

```javascript
function initBuffers()
{
	pyramidVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, pyramidVertexPositionBuffer);
	var vertices = [
					// 正面
					 0.0, 1.0, 0.0,
					-1.0, -1.0, 1.0,
					 1.0, -1.0, 1.0,
					// 右侧面
					 0.0, 1.0, 0.0,
					 1.0, -1.0, 1.0,
					 1.0, -1.0, -1.0,
					// 背面
					 0.0, 1.0, 0.0,
					 1.0, -1.0, -1.0,
					-1.0, -1.0, -1.0,
					// 左侧面
					 0.0, 1.0, 0.0,
					-1.0, -1.0, -1.0,
					-1.0, -1.0, 1.0
					];
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices),
		gl.STATIC_DRAW);
	pyramidVertexPositionBuffer.itemSize = 3;
	pyramidVertexPositionBuffer.numItems = 12;

	pyramidVertexColorBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, pyramidVertexColorBuffer);
	var colors = [
				 // 正面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 1.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 // 右侧面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 0.0, 1.0, 0.0, 1.0,
				 // 背面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 1.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 // 左侧面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 0.0, 1.0, 0.0, 1.0
				];
 	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(colors),
 		gl.STATIC_DRAW);
 	pyramidVertexColorBuffer.itemSize = 4;
 	pyramidVertexColorBuffer.numItems = 12;
```
对四面体，给出了三维世界中每个面的顶点坐标，相应的numItems也自然和三角的不一样了。
```javascript
	cubeVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexPositionBuffer);
	vertices = [
				// 正面
				-1.0, -1.0,  1.0,
				 1.0, -1.0,  1.0,
				 1.0,  1.0,  1.0,
				-1.0,  1.0,  1.0,

				// 背面
				-1.0, -1.0, -1.0,
				-1.0,  1.0, -1.0,
				 1.0,  1.0, -1.0,
				 1.0, -1.0, -1.0,

				// 顶部
				-1.0,  1.0, -1.0,
				-1.0,  1.0,  1.0,
				 1.0,  1.0,  1.0,
				 1.0,  1.0, -1.0,

				// 底部
				-1.0, -1.0, -1.0,
				 1.0, -1.0, -1.0,
				 1.0, -1.0,  1.0,
				-1.0, -1.0,  1.0,

				// 右侧面
				 1.0, -1.0, -1.0,
				 1.0,  1.0, -1.0,
				 1.0,  1.0,  1.0,
				 1.0, -1.0,  1.0,

				// 左侧面
				-1.0, -1.0, -1.0,
				-1.0, -1.0,  1.0,
				-1.0,  1.0,  1.0,
				-1.0,  1.0, -1.0,
				];
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices),
		gl.STATIC_DRAW);
	cubeVertexPositionBuffer.itemSize = 3;
	cubeVertexPositionBuffer.numItems = 24;

	cubeVertexColorBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexColorBuffer);
	colors = [
			  [1.0, 0.0, 0.0, 1.0],	 // 正面
			  [1.0, 1.0, 0.0, 1.0],	 // 背面
			  [0.0, 1.0, 0.0, 1.0],	 // 顶部
			  [1.0, 0.5, 0.5, 1.0],	 // 底部
			  [1.0, 0.0, 1.0, 1.0],	 // 右侧面
			  [0.0, 0.0, 1.0, 1.0]	 // 左侧面
			];
	var unpackedColors = [];
	for (var i in colors)
	{
		var color = colors[i];
		for (var j=0; j < 4; j++)
		{
			unpackedColors = unpackedColors.concat(color);
		}
	}
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(unpackedColors),
		gl.STATIC_DRAW);
	cubeVertexColorBuffer.itemSize = 4;
	cubeVertexColorBuffer.numItems = 24;
```

代码还是把中正方体每个面的顶点分别给出，以便给不同的面设置不同颜色。

颜色按顶点的顺序对应给出，这个for循环代码的逻辑，思考一下应该会明白。

```javascript
	cubeVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	var cubeVertexIndices = [
							  0, 1, 2,	  0, 2, 3,	// 正面
							  4, 5, 6,	  4, 6, 7,	// 背面
							  8, 9, 10,	 8, 10, 11,  // 顶部
							  12, 13, 14,   12, 14, 15, // 底部
							  16, 17, 18,   16, 18, 19, // 右侧面
							  20, 21, 22,   20, 22, 23  // 左侧面
							];
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(cubeVertexIndices), gl.STATIC_DRAW);
	cubeVertexIndexBuffer.itemSize = 1;
	cubeVertexIndexBuffer.numItems = 36;
}
```
索引的设置方法与前面的顶点、颜色类似，只是相应改变了一些参数，如gl.ELEMENT_ARRAY_BUFFER、Uint16Array等。

```javascript
function drawScene()
{
    //...
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	setMatrixUniforms();
	gl.drawElements(gl.TRIANGLES,
		cubeVertexIndexBuffer.numItems, gl.UNSIGNED_SHORT, 0);
	////mvPopMatrix();
}
```
绘制的时候，四面体依然使用gl.drawArrays()直接顶点数组画。对正方体使用索引来绘制，函数相应改为gl.drawElements()，当然这前面要绑定的buffer也要改为IndexBuffer。

### 完整代码
```html
<div class="page-header"><h3>4、来点真正的3D</h3></div>

<canvas id = "test04-canvas" width = "800" height = "600"></canvas>

<script id = "shader-vs" type = "x-shader/x-vertex">
	attribute vec3 aVertexPosition;
	attribute vec4 aVertexColor;
	uniform mat4 uMVMatrix;
	uniform mat4 uPMatrix;
	varying vec4 vColor;
	void main(void)
	{
		gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
		vColor = aVertexColor;
	}
</script>

<script id = "shader-fs" type = "x-shader/x-fragment">
	precision mediump float;

	varying vec4 vColor;

	void main(void)
	{
		gl_FragColor = vColor;
	}
</script>
<script type="text/javascript">
	requestAnimFrame = window.requestAnimationFrame ||
		window.mozRequestAnimationFrame ||
		window.webkitRequestAnimationFrame ||
		window.msRequestAnimationFrame ||
		window.oRequestAnimationFrame ||
		function(callback) { setTimeout(callback, 1000 / 60); };
</script>
<script type="text/javascript">

$(document).ready(function ()
{
	webGLStart();
});

function webGLStart()
{
	var canvas = $("#test04-canvas")[0];
	initGL(canvas);
	initShaders();
	initBuffers();

	gl.clearColor(0.0, 0.0, 0.0, 1.0);
	gl.enable(gl.DEPTH_TEST);

// 	drawScene();

	tick();
}

function tick()
{
	requestAnimFrame(tick);
	drawScene();
	animate();
}

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

function getShader(gl, id)
{
	var shaderScript = $("#" + id);
	if(!shaderScript.length)
	{
		return null;
	}
	var str = shaderScript.text();

	var shader;
	if(shaderScript[0].type == "x-shader/x-fragment")
	{
		shader = gl.createShader(gl.FRAGMENT_SHADER);
	}
	else if(shaderScript[0].type == "x-shader/x-vertex")
	{
		shader = gl.createShader(gl.VERTEX_SHADER);
	}
	else
	{
		return null;
	}
	gl.shaderSource(shader, str);
	gl.compileShader(shader);
	if(!gl.getShaderParameter(shader, gl.COMPILE_STATUS))
	{
		alert(gl.getShaderInfoLog(shader));
		return null;
	}
	return shader;
}

var shaderProgram;
function initShaders()
{
	var fragmentShader = getShader(gl, "shader-fs");
	var vertexShader = getShader(gl, "shader-vs");
	shaderProgram = gl.createProgram();
	gl.attachShader(shaderProgram, vertexShader);
	gl.attachShader(shaderProgram, fragmentShader);
	gl.linkProgram(shaderProgram);

	if(!gl.getProgramParameter(shaderProgram, gl.LINK_STATUS))
	{
		alert("无法初始化“Shader”。");
	}
	gl.useProgram(shaderProgram);

	shaderProgram.vertexPositionAttribute =
		gl.getAttribLocation(shaderProgram, "aVertexPosition");
	gl.enableVertexAttribArray(shaderProgram.vertexPositionAttribute);

	shaderProgram.vertexColorAttribute =
		gl.getAttribLocation(shaderProgram, "aVertexColor");
	gl.enableVertexAttribArray(shaderProgram.vertexColorAttribute);

	shaderProgram.pMatrixUniform =
		gl.getUniformLocation(shaderProgram, "uPMatrix");
	shaderProgram.mvMatrixUniform =
		gl.getUniformLocation(shaderProgram, "uMVMatrix");
}


var mvMatrix = mat4.create();

var mvMatrixStack = [];

var pMatrix = mat4.create();

function mvPushMatrix()
{
	var copy = mat4.clone(mvMatrix);
	mvMatrixStack.push(copy);
}
function mvPopMatrix()
{
	if(mvMatrixStack.length == 0)
	{
		throw "不合法的矩阵出栈操作!";
	}
	mvMatrix = mvMatrixStack.pop();
}

function setMatrixUniforms()
{
	gl.uniformMatrix4fv(shaderProgram.pMatrixUniform, false, pMatrix);
	gl.uniformMatrix4fv(shaderProgram.mvMatrixUniform, false, mvMatrix);
}


var pyramidVertexPositionBuffer;
var pyramidVertexColorBuffer;
var cubeVertexPositionBuffer;
var cubeVertexColorBuffer;
var cubeVertexIndexBuffer;

function initBuffers()
{
	pyramidVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, pyramidVertexPositionBuffer);
	var vertices = [
					// 正面
					 0.0, 1.0, 0.0,
					-1.0, -1.0, 1.0,
					 1.0, -1.0, 1.0,
					// 右侧面
					 0.0, 1.0, 0.0,
					 1.0, -1.0, 1.0,
					 1.0, -1.0, -1.0,
					// 背面
					 0.0, 1.0, 0.0,
					 1.0, -1.0, -1.0,
					-1.0, -1.0, -1.0,
					// 左侧面
					 0.0, 1.0, 0.0,
					-1.0, -1.0, -1.0,
					-1.0, -1.0, 1.0
					];
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices),
		gl.STATIC_DRAW);
	pyramidVertexPositionBuffer.itemSize = 3;
	pyramidVertexPositionBuffer.numItems = 12;

	pyramidVertexColorBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, pyramidVertexColorBuffer);
	var colors = [
				 // 正面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 1.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 // 右侧面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 0.0, 1.0, 0.0, 1.0,
				 // 背面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 1.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 // 左侧面
				 1.0, 0.0, 0.0, 1.0,
				 0.0, 0.0, 1.0, 1.0,
				 0.0, 1.0, 0.0, 1.0
				];
 	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(colors),
 		gl.STATIC_DRAW);
 	pyramidVertexColorBuffer.itemSize = 4;
 	pyramidVertexColorBuffer.numItems = 12;


	cubeVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexPositionBuffer);
	vertices = [
				// 正面
				-1.0, -1.0,  1.0,
				 1.0, -1.0,  1.0,
				 1.0,  1.0,  1.0,
				-1.0,  1.0,  1.0,

				// 背面
				-1.0, -1.0, -1.0,
				-1.0,  1.0, -1.0,
				 1.0,  1.0, -1.0,
				 1.0, -1.0, -1.0,

				// 顶部
				-1.0,  1.0, -1.0,
				-1.0,  1.0,  1.0,
				 1.0,  1.0,  1.0,
				 1.0,  1.0, -1.0,

				// 底部
				-1.0, -1.0, -1.0,
				 1.0, -1.0, -1.0,
				 1.0, -1.0,  1.0,
				-1.0, -1.0,  1.0,

				// 右侧面
				 1.0, -1.0, -1.0,
				 1.0,  1.0, -1.0,
				 1.0,  1.0,  1.0,
				 1.0, -1.0,  1.0,

				// 左侧面
				-1.0, -1.0, -1.0,
				-1.0, -1.0,  1.0,
				-1.0,  1.0,  1.0,
				-1.0,  1.0, -1.0,
				];
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices),
		gl.STATIC_DRAW);
	cubeVertexPositionBuffer.itemSize = 3;
	cubeVertexPositionBuffer.numItems = 24;

	cubeVertexColorBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexColorBuffer);
	colors = [
			  [1.0, 0.0, 0.0, 1.0],	 // 正面
			  [1.0, 1.0, 0.0, 1.0],	 // 背面
			  [0.0, 1.0, 0.0, 1.0],	 // 顶部
			  [1.0, 0.5, 0.5, 1.0],	 // 底部
			  [1.0, 0.0, 1.0, 1.0],	 // 右侧面
			  [0.0, 0.0, 1.0, 1.0]	 // 左侧面
			];
	var unpackedColors = [];
	for (var i in colors)
	{
		var color = colors[i];
		for (var j=0; j < 4; j++)
		{
			unpackedColors = unpackedColors.concat(color);
		}
	}
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(unpackedColors),
		gl.STATIC_DRAW);
	cubeVertexColorBuffer.itemSize = 4;
	cubeVertexColorBuffer.numItems = 24;

	cubeVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	var cubeVertexIndices = [
							  0, 1, 2,	  0, 2, 3,	// 正面
							  4, 5, 6,	  4, 6, 7,	// 背面
							  8, 9, 10,	 8, 10, 11,  // 顶部
							  12, 13, 14,   12, 14, 15, // 底部
							  16, 17, 18,   16, 18, 19, // 右侧面
							  20, 21, 22,   20, 22, 23  // 左侧面
							];
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, new Uint16Array(cubeVertexIndices), gl.STATIC_DRAW);
	cubeVertexIndexBuffer.itemSize = 1;
	cubeVertexIndexBuffer.numItems = 36;
}

var rPyramid = 0;
var rCube = 0;
function drawScene()
{
	gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
	gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

	mat4.perspective(pMatrix, 45, gl.viewportWidth /
		gl.viewportHeight, 0.1, 100.0);

	mat4.identity(mvMatrix);

	mat4.translate(mvMatrix, mvMatrix, [-1.5, 0.0, -8.0]);

	mvPushMatrix();
	mat4.rotate(mvMatrix, mvMatrix, degToRad(rPyramid), [0, 1, 0]);

	gl.bindBuffer(gl.ARRAY_BUFFER, pyramidVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute,
		pyramidVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, pyramidVertexColorBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexColorAttribute,
		pyramidVertexColorBuffer.itemSize, gl.FLOAT, false, 0, 0);

	setMatrixUniforms();
	gl.drawArrays(gl.TRIANGLES, 0,
		pyramidVertexPositionBuffer.numItems);

	mvPopMatrix();

	mat4.translate(mvMatrix, mvMatrix, [ 3.0, 0.0, 0.0]);

	mvPushMatrix();
	mat4.rotate(mvMatrix, mvMatrix, degToRad(rCube), [1, 1, 1]);

	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute,
		cubeVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexColorBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexColorAttribute,
		cubeVertexColorBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	setMatrixUniforms();
	gl.drawElements(gl.TRIANGLES,
		cubeVertexIndexBuffer.numItems, gl.UNSIGNED_SHORT, 0);

	mvPopMatrix();
}
function degToRad(degrees)
{
	return degrees * Math.PI / 180;
}
var lastTime = 0;
function animate()
{
	var timeNow = new Date().getTime();
	if(lastTime != 0)
	{
		var elapsed = timeNow - lastTime;

		rPyramid += (90 * elapsed) / 1000.0;
		rCube += (75 * elapsed) / 1000.0;
	}
	lastTime = timeNow;
}
</script>
```
