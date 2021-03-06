# 镜面地图

```html
<div class="page-header"><h3>15、镜面地图</h3></div>

<canvas id = "test15-canvas" width = "800" height = "600"></canvas>
<br/>
<input type="checkbox" id="color-map" checked /> 使用纹理
<br/>
<input type="checkbox" id="specular-map" checked /> 使用高光标记图
<br/>
<input type="checkbox" id="lighting" checked /> 使用光照
<br/>

<h4>点光:</h4>
<table>
	<tr>
		<td><b>位置:</b></td>
		<td>X: <input type="text" id="lightPositionX" value="-10.0" />
		</td>
		<td>Y: <input type="text" id="lightPositionY" value="4.0" /></td>
		<td>Z: <input type="text" id="lightPositionZ" value="-20.0" /></td>
	</tr>
	<tr>
		<td><b>镜面反射颜色:</b></td>
		<td>R: <input type="text" id="specularR" value="5.0" /></td>
		<td>G: <input type="text" id="specularG" value="5.0" /></td>
		<td>B: <input type="text" id="specularB" value="5.0" /></td>
	</tr>
	<tr>
		<td><b>漫反射颜色:</b>
		<td>R: <input type="text" id="diffuseR" value="0.8" />
		<td>G: <input type="text" id="diffuseG" value="0.8" />
		<td>B: <input type="text" id="diffuseB" value="0.8" />
	</tr>
</table>
<h4>环境光:</h4>
<table style="border: 0; padding: 10px;">
	<tr>
		<td><b>颜色:</b></td>
		<td>R: <input type="text" id="ambientR" value="0.4" /></td>
		<td>G: <input type="text" id="ambientG" value="0.4" /></td>
		<td>B: <input type="text" id="ambientB" value="0.4" /></td>
	</tr>
</table>
<br/>
地球纹理来自： 
<a href="http://www.esa.int/esaEO/SEMGSY2IU7E_index_0.html">
	the European Space Agency/Envisat
</a>.
<br/>

<script id = "per-fragment-lighting-vs" type = "x-shader/x-vertex">
	attribute vec3 aVertexPosition;
	attribute vec3 aVertexNormal;
	attribute vec2 aTextureCoord;

	uniform mat4 uMVMatrix;
	uniform mat4 uPMatrix;
	uniform mat3 uNMatrix;

	varying vec2 vTextureCoord;
	varying vec3 vTransformedNormal;
	varying vec4 vPosition;


	void main(void)
	{
		vPosition = uMVMatrix * vec4(aVertexPosition, 1.0);
		gl_Position = uPMatrix * vPosition;
		vTextureCoord = aTextureCoord;
		vTransformedNormal = uNMatrix * aVertexNormal;
	}
</script>
<script id = "per-fragment-lighting-fs" type = "x-shader/x-fragment">
	precision mediump float;

	varying vec2 vTextureCoord;
	varying vec3 vTransformedNormal;
	varying vec4 vPosition;

	uniform bool uUseColorMap;
	uniform bool uUseSpecularMap;
	uniform bool uUseLighting;

	uniform vec3 uAmbientColor;

	uniform vec3 uPointLightingLocation;
	uniform vec3 uPointLightingSpecularColor;
	uniform vec3 uPointLightingDiffuseColor;

	uniform sampler2D uColorMapSampler;
	uniform sampler2D uSpecularMapSampler;


	void main(void)
	{
		vec3 lightWeighting;
		if (!uUseLighting)
		{
			lightWeighting = vec3(1.0, 1.0, 1.0);
		}
		else
		{
			vec3 lightDirection = 
				normalize(uPointLightingLocation - vPosition.xyz);
			vec3 normal = normalize(vTransformedNormal);

			float specularLightWeighting = 0.0;
			float shininess = 32.0;
			if (uUseSpecularMap)
			{
				shininess = 
					texture2D(uSpecularMapSampler, 
					vec2(vTextureCoord.s, vTextureCoord.t)).r * 255.0;
			}
			if (shininess < 255.0)
			{
				vec3 eyeDirection = normalize(-vPosition.xyz);
				vec3 reflectionDirection = reflect(-lightDirection, normal);

				specularLightWeighting = 
					pow(max(dot(reflectionDirection, eyeDirection), 0.0), 
					shininess);
			}

			float diffuseLightWeighting = max(dot(normal, lightDirection), 0.0);
			lightWeighting = uAmbientColor +
							 uPointLightingSpecularColor * specularLightWeighting +
							 uPointLightingDiffuseColor * diffuseLightWeighting;
		}

		vec4 fragmentColor;
		if (uUseColorMap)
		{
			fragmentColor = 
				texture2D(uColorMapSampler, 
				vec2(vTextureCoord.s, vTextureCoord.t));
		}
		else
		{
			fragmentColor = vec4(1.0, 1.0, 1.0, 1.0);
		}
		gl_FragColor = vec4(fragmentColor.rgb * lightWeighting, fragmentColor.a);
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
	var canvas = $("#test15-canvas");
	initGL(canvas[0]);
	initShaders();
	initTextures();
	initBuffers();

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

var earthColorMapTexture;
var earthSpecularMapTexture;
function initTextures()
{
	earthColorMapTexture = gl.createTexture();
	earthColorMapTexture.image = new Image();
	earthColorMapTexture.image.onload = function ()
	{
		handleLoadedTexture(earthColorMapTexture)
		textureFlag[0] = true;
	}
	earthColorMapTexture.image.src = "/Public/image/earth.jpg";

	earthSpecularMapTexture  = gl.createTexture();
	earthSpecularMapTexture .image = new Image();
	earthSpecularMapTexture .image.onload = function ()
	{
		handleLoadedTexture(earthSpecularMapTexture)
		textureFlag[1] = true;
	}
	earthSpecularMapTexture .image.src = "/Public/image/earth-specular.gif";
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
	var vertexShader = getShader(gl, "per-fragment-lighting-vs");
	var fragmentShader = getShader(gl, "per-fragment-lighting-fs");

	shaderProgram  = gl.createProgram();
	gl.attachShader(shaderProgram, vertexShader);
	gl.attachShader(shaderProgram, fragmentShader);
	gl.linkProgram (shaderProgram );
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
	shaderProgram.colorMapSamplerUniform = 
		gl.getUniformLocation(shaderProgram, "uColorMapSampler");
	shaderProgram.specularMapSamplerUniform = 
		gl.getUniformLocation(shaderProgram, "uSpecularMapSampler");
	shaderProgram.useColorMapUniform = 
		gl.getUniformLocation(shaderProgram, "uUseColorMap");
	shaderProgram.useSpecularMapUniform = 
		gl.getUniformLocation(shaderProgram, "uUseSpecularMap");
	shaderProgram.useLightingUniform = 
		gl.getUniformLocation(shaderProgram, "uUseLighting");
	shaderProgram.ambientColorUniform = 
		gl.getUniformLocation(shaderProgram, "uAmbientColor");
	shaderProgram.pointLightingLocationUniform = 
		gl.getUniformLocation(shaderProgram, "uPointLightingLocation");
	shaderProgram.pointLightingSpecularColorUniform = 
		gl.getUniformLocation(shaderProgram, "uPointLightingSpecularColor");
	shaderProgram.pointLightingDiffuseColorUniform = 
		gl.getUniformLocation(shaderProgram, "uPointLightingDiffuseColor");
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

var sphereVertexPositionBuffer;
var sphereVertexNormalBuffer;
var sphereVertexTextureCoordBuffer;
var sphereVertexIndexBuffer;

function initBuffers()
{
	var latitudeBands = 30;
	var longitudeBands = 30;
	var radius = 13;

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
	sphereVertexNormalBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, sphereVertexNormalBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(normalData), gl.STATIC_DRAW);
	sphereVertexNormalBuffer.itemSize = 3;
	sphereVertexNormalBuffer.numItems = normalData.length / 3;
	
	sphereVertexTextureCoordBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, sphereVertexTextureCoordBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(textureCoordData), gl.STATIC_DRAW);
	sphereVertexTextureCoordBuffer.itemSize = 2;
	sphereVertexTextureCoordBuffer.numItems = textureCoordData.length / 2;

	sphereVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, sphereVertexPositionBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(vertexPositionData), gl.STATIC_DRAW);
	sphereVertexPositionBuffer.itemSize = 3;
	sphereVertexPositionBuffer.numItems = vertexPositionData.length / 3;

	sphereVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, sphereVertexIndexBuffer);
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, 
		new Uint16Array(indexData), gl.STATIC_DRAW);
	sphereVertexIndexBuffer.itemSize = 1;
	sphereVertexIndexBuffer.numItems = indexData.length;
}

var earthAngle = 180;

function drawScene()
{
	gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
	gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

	mat4.perspective(pMatrix, 45, 
		gl.viewportWidth / gl.viewportHeight, 0.1, 100.0);

	var useColorMap  = $("#color-map").is(":checked");
	gl.uniform1i(shaderProgram.useColorMapUniform, useColorMap);

	var useSpecularMap = $("#specular-map").is(":checked");
	gl.uniform1i(shaderProgram.useSpecularMapUniform, useSpecularMap );

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
			shaderProgram.pointLightingSpecularColorUniform,
			parseFloat($("#specularR").val()),
			parseFloat($("#specularG").val()),
			parseFloat($("#specularB").val())
		);

		gl.uniform3f(
			shaderProgram.pointLightingDiffuseColorUniform,
			parseFloat($("#diffuseR").val()),
			parseFloat($("#diffuseG").val()),
			parseFloat($("#diffuseB").val())
		);

	}

	mat4.identity(mvMatrix);
	mat4.translate(mvMatrix, mvMatrix, [0, 0, -40]);
	mat4.rotate(mvMatrix, mvMatrix, degToRad(23.4), [1, 0, -1]);
	mat4.rotate(mvMatrix, mvMatrix, degToRad(earthAngle), [0, 1, 0]);

	gl.activeTexture(gl.TEXTURE0);
	gl.bindTexture(gl.TEXTURE_2D, earthColorMapTexture);
	gl.uniform1i(shaderProgram.specularMapSamplerUniform, 0);

	gl.activeTexture(gl.TEXTURE1);
	gl.bindTexture(gl.TEXTURE_2D, earthSpecularMapTexture);
	gl.uniform1i(shaderProgram.specularMapSamplerUniform, 1);

	gl.bindBuffer(gl.ARRAY_BUFFER, sphereVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, 
		sphereVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, sphereVertexNormalBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexNormalAttribute, 
		sphereVertexNormalBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, sphereVertexTextureCoordBuffer);
	gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, 
		sphereVertexTextureCoordBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, sphereVertexIndexBuffer);
	setMatrixUniforms();
	gl.drawElements(gl.TRIANGLES, 
		sphereVertexIndexBuffer.numItems, gl.UNSIGNED_SHORT, 0);
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

	 	earthAngle += 0.05 * elapsed;
	}
	lastTime = timeNow;
}
</script>
```