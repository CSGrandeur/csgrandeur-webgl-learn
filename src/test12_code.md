# 点光源

```html
<div class="page-header"><h3>12、点光源</h3></div>

<canvas id = "test12-canvas" width = "800" height = "600"></canvas>

<br/>
<input type="checkbox" id="lighting" checked /> 使用光照
<br/>
<h4>点光:</h4>
<table>
	<tr>
		<td><b>位置:</b></td>
		<td>X: <input type="text" id="lightPositionX" value="0.0" /></td>
		<td>Y: <input type="text" id="lightPositionY" value="0.0" /></td>
		<td>Z: <input type="text" id="lightPositionZ" value="-20.0" /></td>
	</tr>
	<tr>
		<td><b>颜色:</b></td>
		<td>R: <input type="text" id="pointR" value="0.8" /></td>
		<td>G: <input type="text" id="pointG" value="0.8" /></td>
		<td>B: <input type="text" id="pointB" value="0.8" /></td>
	</tr>
</table>
<h4>环境光:</h4>
<table style="border: 0; padding: 10px;">
	<tr>
		<td><b>颜色:</b></td>
		<td>R: <input type="text" id="ambientR" value="0.2" /></td>
		<td>G: <input type="text" id="ambientG" value="0.2" /></td>
		<td>B: <input type="text" id="ambientB" value="0.2" /></td>
	</tr>
</table>
<br/>
月球表面纹理图片来自 
<a href="http://maps.jpl.nasa.gov/">the Jet Propulsion Laboratory</a>.
<script id = "shader-vs" type = "x-shader/x-vertex">
	attribute vec3 aVertexPosition;
	attribute vec3 aVertexNormal;
	attribute vec2 aTextureCoord;

	uniform mat4 uMVMatrix;
	uniform mat4 uPMatrix;
	uniform mat3 uNMatrix;

	uniform vec3 uAmbientColor;

	uniform vec3 uPointLightingLocation;
	uniform vec3 uPointLightingColor;

	uniform bool uUseLighting;

	varying vec2 vTextureCoord;
	varying vec3 vLightWeighting;

	void main(void)
	{
	    vec4 mvPosition = uMVMatrix * vec4(aVertexPosition, 1.0);
	    gl_Position = uPMatrix * mvPosition;
	    vTextureCoord = aTextureCoord;

	    if (!uUseLighting)
	    {
	        vLightWeighting = vec3(1.0, 1.0, 1.0);
	    }
	    else
	    {
	        vec3 lightDirection = 
	        	normalize(uPointLightingLocation - mvPosition.xyz);
	        vec3 transformedNormal = uNMatrix * aVertexNormal;
	        float directionalLightWeighting = 
	        	max(dot(transformedNormal, lightDirection), 0.0);
	        vLightWeighting = 
	        	uAmbientColor + uPointLightingColor * directionalLightWeighting;
	    }
	}
</script>

<script id = "shader-fs" type = "x-shader/x-fragment">
	precision mediump float;

	varying vec2 vTextureCoord;
	varying vec3 vLightWeighting;

	uniform sampler2D uSampler;

	void main(void)
	{
	    vec4 textureColor = 
	    	texture2D(uSampler, vec2(vTextureCoord.s, vTextureCoord.t));
	    gl_FragColor = 
	    	vec4(textureColor.rgb * vLightWeighting, textureColor.a);
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
	var canvas = $("#test12-canvas");
	initGL(canvas[0]);
	initShaders();
	initBuffers();
	initTexture();

	gl.clearColor(0.0, 0.0, 0.0, 1.0);
	gl.enable(gl.DEPTH_TEST);
	
	waitTexture = setInterval("tick()", 100);
}

var textureFlag = [0, 0];
var startTick = false;
var waitTick;
function tick()
{
	if(textureFlag[0] == 1 && textureFlag[1] == 1)
	{
		startTick = true;
		textureFlag[0] = 2;
		textureFlag[1] = 2;
		clearInterval(waitTexture);
	}
	if(startTick)
	{
		requestAnimFrame(tick);
		drawScene();
		animate();
	}
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

var moonTexture;
var crateTexture;
function initTexture()
{
	moonTexture = gl.createTexture();
	moonTexture.image = new Image();
	moonTexture.image.onload = function()
	{
		handleLoadedTexture(moonTexture);
		textureFlag[0] = true;
	}
	moonTexture.image.src = "/Public/image/moon.gif";


	crateTexture = gl.createTexture();
	crateTexture.image = new Image();
	crateTexture.image.onload = function()
	{
		handleLoadedTexture(crateTexture);
		textureFlag[1] = true;
	}
	crateTexture.image.src = "/Public/image/crate.gif";
}
function handleLoadedTexture(texture)
{
	gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, 1);
	

	gl.bindTexture(gl.TEXTURE_2D, texture);
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, 
		gl.UNSIGNED_BYTE, texture.image);
	gl.texParameteri(gl.TEXTURE_2D, 
		gl.TEXTURE_MAG_FILTER, gl.LINEAR);
	gl.texParameteri(gl.TEXTURE_2D, 
		gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_NEAREST);

	gl.generateMipmap(gl.TEXTURE_2D);
	
	gl.bindTexture(gl.TEXTURE_2D, null);
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

	shaderProgram.vertexNormalAttribute = 
		gl.getAttribLocation(shaderProgram, "aVertexNormal");
	gl.enableVertexAttribArray(shaderProgram.vertexNormalAttribute);

	shaderProgram.textureCoordAttribute = 
		gl.getAttribLocation(shaderProgram, "aTextureCoord");
	gl.enableVertexAttribArray(shaderProgram.textureCoordAttribute);

	shaderProgram.pMatrixUniform = 
		gl.getUniformLocation(shaderProgram, "uPMatrix");
	shaderProgram.mvMatrixUniform = 
		gl.getUniformLocation(shaderProgram, "uMVMatrix");
	shaderProgram.nMatrixUniform  = 
		gl.getUniformLocation(shaderProgram, "uNMatrix");
	shaderProgram.samplerUniform = 
		gl.getUniformLocation(shaderProgram, "uSampler");
	shaderProgram.useLightingUniform = 
		gl.getUniformLocation(shaderProgram, "uUseLighting");
	shaderProgram.ambientColorUniform = 
		gl.getUniformLocation(shaderProgram, "uAmbientColor");
	shaderProgram.pointLightingLocationUniform  = 
		gl.getUniformLocation(shaderProgram, "uPointLightingLocation");
	shaderProgram.pointLightingColorUniform  = 
		gl.getUniformLocation(shaderProgram, "uPointLightingColor");
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

	var normalMatrix = mat3.create();

	mat3.fromMat4(normalMatrix, mvMatrix);
	mat3.invert(normalMatrix, normalMatrix);
	mat3.transpose(normalMatrix, normalMatrix);

	gl.uniformMatrix3fv(shaderProgram.nMatrixUniform, false, normalMatrix);
}

var cubeVertexPositionBUffer;
var cubeVertexTextureCoordBuffer;
var cubeVertexIndexBuffer;
var cubeVertexNormalBuffer;

var moonVertexPositionBuffer;
var moonVertexNormalBuffer;
var moonVertexTextureCoordBuffer;
var moonVertexIndexBuffer;

function initBuffers()
{
	//箱子
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
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(vertices), gl.STATIC_DRAW);
	cubeVertexPositionBuffer.itemSize = 3;
	cubeVertexPositionBuffer.numItems = 24;

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
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(textureCoords), gl.STATIC_DRAW);
	cubeVertexTextureCoordBuffer.itemSize = 2;
	cubeVertexTextureCoordBuffer.numItems = 24;

	cubeVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	var cubeVertexIndices = 
		[
		  0, 1, 2,		0, 2, 3,	// 正面
		  4, 5, 6,		4, 6, 7,	// 背面
		  8, 9, 10,		8, 10, 11,  // 顶部
		  12, 13, 14,	12, 14, 15, // 底部
		  16, 17, 18,	16, 18, 19, // 右侧面
		  20, 21, 22,	20, 22, 23  // 左侧面
		];
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, 
		new Uint16Array(cubeVertexIndices), gl.STATIC_DRAW);
	cubeVertexIndexBuffer.itemSize = 1;
	cubeVertexIndexBuffer.numItems = 36;

	//月亮
		var latitudeBands = 30;
	var longitudeBands = 30;
	var radius = 2;
	var vertexPositionData = [];
	var normalData = [];
	var textureCoordData = [];
	for(var latNumber = 0; latNumber <= latitudeBands; latNumber ++)
	{
		var theta = latNumber * Math.PI / latitudeBands;
		var sinTheta = Math.sin(theta);
		var cosTheta = Math.cos(theta);
		for(var longNumber = 0; longNumber <= longitudeBands; longNumber ++)
		{
			var phi = longNumber * 2 * Math.PI / longitudeBands;
			var sinPhi = Math.sin(phi);
			var cosPhi = Math.cos(phi);
			var x = cosPhi * sinTheta;
			var y = cosTheta;
			var z = sinPhi * sinTheta;
			var u = 1 - (longNumber / longitudeBands);
			var v = 1 - (latNumber / latitudeBands);

			normalData.push(x);
			normalData.push(y);
			normalData.push(z);
			textureCoordData.push(u);
			textureCoordData.push(v);
			vertexPositionData.push(radius * x);
			vertexPositionData.push(radius * y);
			vertexPositionData.push(radius * z);
		}
	}
	var indexData = [];
	for(var latNumber = 0; latNumber < latitudeBands; latNumber ++)
	{
		for(var longNumber = 0; longNumber < longitudeBands; longNumber ++)
		{
			var first = (latNumber * (longitudeBands + 1)) + longNumber;
			var second = first + longitudeBands + 1;
			indexData.push(first);
			indexData.push(second);
			indexData.push(first + 1);
			indexData.push(second);
			indexData.push(second + 1);
			indexData.push(first + 1);
		}
	}
	moonVertexNormalBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexNormalBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(normalData), gl.STATIC_DRAW);
	moonVertexNormalBuffer.itemSize = 3;
	moonVertexNormalBuffer.numItems = normalData.length / 3;
	
	moonVertexTextureCoordBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexTextureCoordBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(textureCoordData), gl.STATIC_DRAW);
	moonVertexTextureCoordBuffer.itemSize = 2;
	moonVertexTextureCoordBuffer.numItems = textureCoordData.length / 2;

	moonVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexPositionBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(vertexPositionData), gl.STATIC_DRAW);
	moonVertexPositionBuffer.itemSize = 3;
	moonVertexPositionBuffer.numItems = vertexPositionData.length / 3;

	moonVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, moonVertexIndexBuffer);
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, 
		new Uint16Array(indexData), gl.STATIC_DRAW);
	moonVertexIndexBuffer.itemSize = 1;
	moonVertexIndexBuffer.numItems = indexData.length;
}

var moonAngle = 180;
var cubeAngle = 0;

function drawScene()
{
	gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
	gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

	mat4.perspective(pMatrix, 45, 
		gl.viewportWidth / gl.viewportHeight, 0.1, 100.0);

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
		gl.uniform3f(
			shaderProgram.pointLightingLocationUniform,
			parseFloat($("#lightPositionX").val()),
			parseFloat($("#lightPositionY").val()),
			parseFloat($("#lightPositionZ").val())
		);

		gl.uniform3f(
			shaderProgram.pointLightingColorUniform,
			parseFloat($("#pointR").val()),
			parseFloat($("#pointG").val()),
			parseFloat($("#pointB").val())
		);

	}

	mat4.identity(mvMatrix);
	mat4.translate(mvMatrix, mvMatrix, [0, 0, -20]);

	mvPushMatrix();

	mat4.rotate(mvMatrix, mvMatrix, degToRad(moonAngle), [0, 1, 0]);
	mat4.translate(mvMatrix, mvMatrix, [5, 0, 0]);

	gl.activeTexture(gl.TEXTURE0);
	gl.bindTexture(gl.TEXTURE_2D, moonTexture);
	gl.uniform1i(shaderProgram.samplerUniform, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexTextureCoordBuffer);
	gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, 
		moonVertexTextureCoordBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, 
		moonVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexNormalBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexNormalAttribute, 
		moonVertexNormalBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, moonVertexIndexBuffer);
	setMatrixUniforms();
	gl.drawElements(gl.TRIANGLES, 
		moonVertexIndexBuffer.numItems, gl.UNSIGNED_SHORT, 0);

	mvPopMatrix();

	mvPushMatrix();
	mat4.rotate(mvMatrix, mvMatrix, degToRad(cubeAngle), [0, 1, 0]);
	mat4.translate(mvMatrix, mvMatrix, [5, 0, 0]);

	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, 
		cubeVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexNormalBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexNormalAttribute, 
		cubeVertexNormalBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexTextureCoordBuffer);
	gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, 
		cubeVertexTextureCoordBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.activeTexture(gl.TEXTURE0);
	gl.bindTexture(gl.TEXTURE_2D, crateTexture);
	gl.uniform1i(shaderProgram.samplerUniform, 0);

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

	 	moonAngle += 0.05 * elapsed;
	 	cubeAngle += 0.05 * elapsed;
	}
	lastTime = timeNow;
}
</script>
```