# 键盘交互与纹理过滤
```html
<div class="page-header"><h3>6、键盘输入与纹理过滤</h3></div>

<canvas id = "test06-canvas" width = "800" height = "600"></canvas>
<li>逗号/句号 控制 靠近/远离</li>
<li>WSAD键控制四个方向旋转速度</li>
<li>F键控制纹理过滤类型切换</li>
<script id = "shader-vs" type = "x-shader/x-vertex">
	attribute vec3 aVertexPosition;
	attribute vec2 aTextureCoord;

	uniform mat4 uMVMatrix;
	uniform mat4 uPMatrix;

	varying vec2 vTextureCoord;
	void main(void)
	{
		gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
		vTextureCoord = aTextureCoord;
	}
</script>

<script id = "shader-fs" type = "x-shader/x-fragment">
	precision mediump float;
	varying vec2 vTextureCoord;
	uniform sampler2D uSampler;
	void main(void)
	{
		gl_FragColor = 
			texture2D(uSampler, vec2(vTextureCoord.s, vTextureCoord.t));
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
	var canvas = $("#test06-canvas")[0];
	initGL(canvas);
	initShaders();
	initBuffers();
	initTexture();

	gl.clearColor(0.0, 0.0, 0.0, 1.0);
	gl.enable(gl.DEPTH_TEST);
	
//	setTimeout("tick()", 100);

	$(document).keydown(handleKeyDown);
	$(document).keyup(handleKeyUp);
}
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
		tick();
	}
	crateImage.src = "/Public/image/crate.gif";
}
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
		gl.UNSIGNED_BYTE, textures[1].image);
	gl.texParameteri(gl.TEXTURE_2D, 
		gl.TEXTURE_MAG_FILTER, gl.LINEAR);
	gl.texParameteri(gl.TEXTURE_2D, 
		gl.TEXTURE_MIN_FILTER, gl.LINEAR);

	gl.bindTexture(gl.TEXTURE_2D, textures[2]);
	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, 
		gl.UNSIGNED_BYTE, textures[2].image);
	gl.texParameteri(gl.TEXTURE_2D, 
		gl.TEXTURE_MAG_FILTER, gl.LINEAR);
	gl.texParameteri(gl.TEXTURE_2D, 
		gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_NEAREST);

	gl.generateMipmap(gl.TEXTURE_2D);
	
	gl.bindTexture(gl.TEXTURE_2D, null);
}
function tick()
{
	requestAnimFrame(tick);
	handleKeys();
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

	shaderProgram.textureCoordAttribute = 
		gl.getAttribLocation(shaderProgram, "aTextureCoord");
	gl.enableVertexAttribArray(shaderProgram.textureCoordAttribute);

	shaderProgram.pMatrixUniform = 
		gl.getUniformLocation(shaderProgram, "uPMatrix");
	shaderProgram.mvMatrixUniform = 
		gl.getUniformLocation(shaderProgram, "uMVMatrix");
	shaderProgram.samplerUniform = 
		gl.getUniformLocation(shaderProgram, "uSampler");
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


var cubeVertexPositionBUffer;
var cubeVertexTextureCoordBuffer;
var cubeVertexIndexBuffer;

function initBuffers()
{
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
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(textureCoords), 
		gl.STATIC_DRAW);
	cubeVertexTextureCoordBuffer.itemSize = 2;
	cubeVertexTextureCoordBuffer.numItems = 24;
	
	cubeVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	var cubeVertexIndices = 
		[
		  0, 1, 2,	  0, 2, 3,	// 正面
		  4, 5, 6,	  4, 6, 7,	// 背面
		  8, 9, 10,	 8, 10, 11,  // 顶部
		  12, 13, 14,   12, 14, 15, // 底部
		  16, 17, 18,   16, 18, 19, // 右侧面
		  20, 21, 22,   20, 22, 23  // 左侧面
		];
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, 
		new Uint16Array(cubeVertexIndices), gl.STATIC_DRAW);
	cubeVertexIndexBuffer.itemSize = 1;
	cubeVertexIndexBuffer.numItems = 36;
}

var xRot = 0;
var xSpeed = 0;

var yRot = 0;
var ySpeed = 0;

var z = -5.0;

var filter = 0;

var currentlyPressedKeys = {};
function handleKeyDown(event)
{
	currentlyPressedKeys[event.keyCode] = true;
	if(String.fromCharCode(event.keyCode) == "F")
	{
		filter += 1;
		if(filter == 3)
		{
			filter = 0;
		}
	}
}
function handleKeyUp(event)
{
	currentlyPressedKeys[event.keyCode] = false;
}
function handleKeys()
{
	if(currentlyPressedKeys[190])
	{
		//"."/">"句号键
		z -= 0.05;
	}
	if(currentlyPressedKeys[188])
	{
		//","/"<"逗号键
		z += 0.05;
	}
	if(currentlyPressedKeys[65])
	{
		//A
		ySpeed -= 1;
	}
	if(currentlyPressedKeys[68])
	{
		//D
		ySpeed += 1;
	}
	if(currentlyPressedKeys[87])
	{
		//W
		xSpeed -= 1;
	}
	if(currentlyPressedKeys[83])
	{
		//S
		xSpeed += 1;
	}	
}
function drawScene()
{
	gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
	gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

	mat4.perspective(pMatrix, 45, gl.viewportWidth / 
		gl.viewportHeight, 0.1, 100.0);

	mat4.identity(mvMatrix);

	mat4.translate(mvMatrix, mvMatrix, [0.0, 0.0, z]);

	mat4.rotate(mvMatrix, mvMatrix, degToRad(xRot), [1, 0, 0]);
	mat4.rotate(mvMatrix, mvMatrix, degToRad(yRot), [0, 1, 0]);
	
	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, 
		cubeVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, cubeVertexTextureCoordBuffer);
	gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, 
		cubeVertexTextureCoordBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.activeTexture(gl.TEXTURE0);
	gl.bindTexture(gl.TEXTURE_2D, crateTextures[filter]);
	gl.uniform1i(shaderProgram.samplerUniform, 0);
	
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	setMatrixUniforms();
	gl.drawElements(gl.TRIANGLES, 
		cubeVertexIndexBuffer.numItems, gl.UNSIGNED_SHORT, 0);
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

		xRot += (xSpeed * elapsed) / 1000.0;
		yRot += (ySpeed * elapsed) / 1000.0;
	}
	lastTime = timeNow;
}
</script>
```