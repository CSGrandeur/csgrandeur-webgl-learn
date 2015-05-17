# 球、旋转矩阵、鼠标事件

```html
<div class="page-header"><h3>11、球、旋转矩阵、鼠标事件</h3></div>

<canvas id = "test11-canvas" width = "800" height = "600"></canvas>

<br/>
<input type="checkbox" id="lighting" checked /> 使用光照
<br/>
(鼠标拖拽对月球进行转动)
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
	var canvas = $("#test11-canvas");
	initGL(canvas[0]);
	initShaders();
	initBuffers();
	initTexture();

	gl.clearColor(0.0, 0.0, 0.0, 1.0);
	gl.enable(gl.DEPTH_TEST);

	canvas.mousedown(handleMouseDown);
	$(document).mouseup(handleMouseUp);
	$(document).mousemove(handleMouseMove);
}
function tick()
{
	requestAnimFrame(tick);
	drawScene();
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
function initTexture()
{
	moonTexture = gl.createTexture();
	moonTexture.image = new Image();
	moonTexture.image.onload = function()
	{
		handleLoadedTexture(moonTexture);
		tick();
	}
	moonTexture.image.src = "/Public/image/moon.gif";
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

var mouseDown = false;
var lastMouseX = null;
var lastMouseY = null;
var moonRotationMatrix = mat4.create();
mat4.identity(moonRotationMatrix);
function handleMouseDown(event)
{
	mouseDown = true;
	lastMouseX = event.clientX;
	lastMouseY = event.clientY;
}
function handleMouseUp(event)
{
	mouseDown = false;
}
function handleMouseMove(event)
{
	if(!mouseDown)
	{
		return;
	}
	var newX = event.clientX;
	var newY = event.clientY;
	var newRotationMatrix = mat4.create();
	mat4.identity(newRotationMatrix);

	var deltaX = newX - lastMouseX;
	mat4.rotate(newRotationMatrix, newRotationMatrix, 
		degToRad(deltaX / 10), [0, 1, 0]);
	var deltaY = newY - lastMouseY;
	mat4.rotate(newRotationMatrix, newRotationMatrix, 
		degToRad(deltaY / 10), [1, 0, 0]);
	
	mat4.multiply(moonRotationMatrix, newRotationMatrix, moonRotationMatrix);

	lastMouseX = newX;
	lastMouseY = newY;
}
function equal(a, b)
{
	for(var i = 0; i < a.length; i ++)
		if(a[i] != b[i]) return false;
	return true;
}
var moonVertexPositionBuffer;
var moonVertexNormalBuffer;
var moonVertexTextureCoordBuffer;
var moonVertexIndexBuffer;

function initBuffers()
{
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

	mat4.identity(mvMatrix);
	mat4.translate(mvMatrix, mvMatrix, [0, 0, -6]);

	mat4.multiply(mvMatrix, mvMatrix, moonRotationMatrix);

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
}

function degToRad(degrees)
{
	return degrees * Math.PI / 180;
}
</script>
```