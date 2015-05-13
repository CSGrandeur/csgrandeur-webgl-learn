# 深度缓冲，透明度与混合
```html
<div class="page-header"><h3>8、深度缓冲，透明度与混合</h3></div>

<canvas id = "test08-canvas" width = "800" height = "600"></canvas>
<br/>
<input type="checkbox" id="blending" checked /> 使用混合
<br/>
不透明度（Alpha）级别 <input type="text" id="alpha" value="0.5" /><br/>
<br/>
<input type="checkbox" id="lighting" checked /> 使用光照
<br/>
(逗号/句号 控制 靠近/远离，WSAD键控制四个方向旋转速度)
<br/>
<h4>方向光:</h4>
<table>
	<tr>
		<td><b>方向:</b></td>
		<td>X: <input type="text" id="lightDirectionX" value="-0.25" /></td>
		<td>Y: <input type="text" id="lightDirectionY" value="-0.25" /></td>
		<td>Z: <input type="text" id="lightDirectionZ" value="-1.0" /></td>
	</tr>
	<tr>
		<td><b>颜色:</b></td>
		<td>R: <input type="text" id="directionalR" value="0.8" /></td>
		<td>G: <input type="text" id="directionalG" value="0.8" /></td>
		<td>B: <input type="text" id="directionalB" value="0.8" /></td>
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

<script id = "shader-vs" type = "x-shader/x-vertex">
	attribute vec3 aVertexPosition;
	attribute vec3 aVertexNormal;
	attribute vec2 aTextureCoord;

	uniform mat4 uMVMatrix;
	uniform mat4 uPMatrix;
	uniform mat3 uNMatrix;

	uniform vec3 uAmbientColor;

	uniform vec3 uLightingDirection;
	uniform vec3 uDirectionalColor;

	uniform bool uUseLighting;

	varying vec2 vTextureCoord;
	varying vec3 vLightWeighting;

	void main(void)
	{
		gl_Position = uPMatrix * uMVMatrix * vec4(aVertexPosition, 1.0);
		vTextureCoord = aTextureCoord;

		if(!uUseLighting)
		{
			vLightWeighting = vec3(1.0, 1.0, 1.0);
		}
		else
        {
            vec3 transformedNormal = uNMatrix * aVertexNormal;
            float directionalLightWeighting = 
                max(dot(transformedNormal, uLightingDirection), 0.0);
            vLightWeighting = 
                uAmbientColor + uDirectionalColor * directionalLightWeighting;
		}
	}
</script>

<script id = "shader-fs" type = "x-shader/x-fragment">
	precision mediump float;
	varying vec2 vTextureCoord;
	varying vec3 vLightWeighting;

	uniform float uAlpha;

	uniform sampler2D uSampler;
	void main(void)
	{
		vec4 textureColor = 
			texture2D(uSampler, vec2(vTextureCoord.s, vTextureCoord.t));
		gl_FragColor = 
			vec4(textureColor.rgb * vLightWeighting, textureColor.a * uAlpha);
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
	var canvas = $("#test08-canvas")[0];
	initGL(canvas);
	initShaders();
	initBuffers();
	initTexture();

	gl.clearColor(0.0, 0.0, 0.0, 1.0);
	gl.enable(gl.DEPTH_TEST);
//  gl.depthFunc(gl.LESS);
//	setTimeout("tick()", 100);

	$(document).keydown(handleKeyDown);
	$(document).keyup(handleKeyUp);
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

var crateTexture;
function initTexture()
{
	crateTexture = gl.createTexture();
	crateTexture.image = new Image();
	crateTexture.image.onload = function()
	{
		handleLoadedTexture(crateTexture);
		tick();
	}
	crateTexture.image.src = "/Public/image/glass.gif";
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
    shaderProgram.lightingDirectionUniform = 
    	gl.getUniformLocation(shaderProgram, "uLightingDirection");
    shaderProgram.directionalColorUniform = 
    	gl.getUniformLocation(shaderProgram, "uDirectionalColor");
    shaderProgram.alphaUniform = 
    	gl.getUniformLocation(shaderProgram, "uAlpha");
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
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertexNormals), gl.STATIC_DRAW);
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
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(textureCoords), 
		gl.STATIC_DRAW);
	cubeVertexTextureCoordBuffer.itemSize = 2;
	cubeVertexTextureCoordBuffer.numItems = 24;
	
	cubeVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, cubeVertexIndexBuffer);
	var cubeVertexIndices = 
		[
		  0, 1, 2,	    0, 2, 3,	// 正面
		  4, 5, 6,	    4, 6, 7,	// 背面
		  8, 9, 10,	    8, 10, 11,  // 顶部
		  12, 13, 14,	12, 14, 15, // 底部
		  16, 17, 18,	16, 18, 19, // 右侧面
		  20, 21, 22,	20, 22, 23  // 左侧面
		];
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, 
		new Uint16Array(cubeVertexIndices), gl.STATIC_DRAW);
	cubeVertexIndexBuffer.itemSize = 1;
	cubeVertexIndexBuffer.numItems = 36;


}

var xRot = 0;
var xSpeed = 3;

var yRot = 0;
var ySpeed = -3;

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

	mat4.perspective(pMatrix, 45, 
		gl.viewportWidth / gl.viewportHeight, 0.1, 100.0);

	mat4.identity(mvMatrix);

	mat4.translate(mvMatrix, mvMatrix, [0.0, 0.0, z]);

	mat4.rotate(mvMatrix, mvMatrix, degToRad(xRot), [1, 0, 0]);
	mat4.rotate(mvMatrix, mvMatrix, degToRad(yRot), [0, 1, 0]);
	
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

	var blending = $("#blending").is(":checked");
	if(blending)
	{
		gl.blendFunc(gl.SRC_ALPHA, gl.ONE);
		gl.enable(gl.BLEND);
		gl.disable(gl.DEPTH_TEST);
		gl.uniform1f(shaderProgram.alphaUniform, parseFloat($("#alpha").val()));
	}
	else
	{
		gl.disable(gl.BLEND);
		gl.enable(gl.DEPTH_TEST);
		gl.uniform1f(shaderProgram.alphaUniform, 1);
	}

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