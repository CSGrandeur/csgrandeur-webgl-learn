# 渲染到纹理

```html
<div class="page-header"><h3>16、渲染到纹理</h3></div>

<canvas id = "test16-canvas" width = "800" height = "600"></canvas>
<br/>
笔记本模型来自： 
<a href="http://www.turbosquid.com/3d-models/apple-macbook-max-free/391534">
	this 3DS Max model by Xedium
</a>
<br/>
月球表面纹理图片来自：
<a href="http://maps.jpl.nasa.gov/">the Jet Propulsion Laboratory</a>.
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

	uniform vec3 uMaterialAmbientColor;
	uniform vec3 uMaterialDiffuseColor;
	uniform vec3 uMaterialSpecularColor;
	uniform float uMaterialShininess;
	uniform vec3 uMaterialEmissiveColor;

	uniform bool uShowSpecularHighlights;
	uniform bool uUseTextures;

	uniform vec3 uAmbientLightingColor;

	uniform vec3 uPointLightingLocation;
	uniform vec3 uPointLightingDiffuseColor;
	uniform vec3 uPointLightingSpecularColor;

	uniform sampler2D uSampler;


	void main(void)
	{
		vec3 ambientLightWeighting = uAmbientLightingColor;

		vec3 lightDirection = normalize(uPointLightingLocation - vPosition.xyz);
		vec3 normal = normalize(vTransformedNormal);

		vec3 specularLightWeighting = vec3(0.0, 0.0, 0.0);
		if (uShowSpecularHighlights)
		{
			vec3 eyeDirection = normalize(-vPosition.xyz);
			vec3 reflectionDirection = reflect(-lightDirection, normal);

			float specularLightBrightness = 
				pow(max(dot(reflectionDirection, eyeDirection), 0.0), 
				uMaterialShininess);
			specularLightWeighting = 
				uPointLightingSpecularColor * specularLightBrightness;
		}

		float diffuseLightBrightness = max(dot(normal, lightDirection), 0.0);
		vec3 diffuseLightWeighting = 
			uPointLightingDiffuseColor * diffuseLightBrightness;

		vec3 materialAmbientColor = uMaterialAmbientColor;
		vec3 materialDiffuseColor = uMaterialDiffuseColor;
		vec3 materialSpecularColor = uMaterialSpecularColor;
		vec3 materialEmissiveColor = uMaterialEmissiveColor;
		float alpha = 1.0;
		if (uUseTextures)
		{
			vec4 textureColor = 
				texture2D(uSampler, vec2(vTextureCoord.s, vTextureCoord.t));
			materialAmbientColor = materialAmbientColor * textureColor.rgb;
			materialDiffuseColor = materialDiffuseColor * textureColor.rgb;
			materialEmissiveColor = materialEmissiveColor * textureColor.rgb;
			alpha = textureColor.a;
		}
		gl_FragColor = vec4(
						   materialAmbientColor * ambientLightWeighting +
						   materialDiffuseColor * diffuseLightWeighting +
						   materialSpecularColor * specularLightWeighting +
						   materialEmissiveColor,
						   alpha
					   );
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
	var canvas = $("#test16-canvas");
	initGL(canvas[0]);
	initTextureFramebuffer();
	initShaders();
	initTextures();
	initBuffers();

	loadLaptop();

	gl.clearColor(0.0, 0.0, 0.0, 1.0);
	gl.enable(gl.DEPTH_TEST);
	
	waitTexture = setInterval("tick()", 100);
}

var textureFlag = [0, 0, 0];
var startTick = false;
var waitTick;
function tick()
{
	if(textureFlag[0] == 1 && textureFlag[1] == 1 && textureFlag[2] == 1)
	{
		startTick = true;
		textureFlag[0] = 2;
		textureFlag[1] = 2;
		textureFlag[2] = 2;
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
var rttFramebuffer;
var rttTexture;
function initTextureFramebuffer()
{
	rttFramebuffer = gl.createFramebuffer();
	gl.bindFramebuffer(gl.FRAMEBUFFER, rttFramebuffer);
	rttFramebuffer.width = 512;
	rttFramebuffer.height = 512;

	rttTexture = gl.createTexture();
	gl.bindTexture(gl.TEXTURE_2D, rttTexture);
	gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
	gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
	// gl.generateMipmap(gl.TEXTURE_2D);

	gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, 
		rttFramebuffer.width, rttFramebuffer.height, 
		0, gl.RGBA, gl.UNSIGNED_BYTE, null);

	var renderbuffer = gl.createRenderbuffer();
	gl.bindRenderbuffer(gl.RENDERBUFFER, renderbuffer);
	gl.renderbufferStorage(gl.RENDERBUFFER, gl.DEPTH_COMPONENT16, 
		rttFramebuffer.width, rttFramebuffer.height);

	gl.framebufferTexture2D(
		gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, rttTexture, 0);
	gl.framebufferRenderbuffer(
		gl.FRAMEBUFFER, gl.DEPTH_ATTACHMENT, gl.RENDERBUFFER, renderbuffer);

	gl.bindTexture(gl.TEXTURE_2D, null);
	gl.bindRenderbuffer(gl.RENDERBUFFER, null);
	gl.bindFramebuffer(gl.FRAMEBUFFER, null);
}
var moonTexture;
var crateTexture;
function initTextures()
{
	moonTexture = gl.createTexture();
	moonTexture.image = new Image();
	moonTexture.image.onload = function ()
	{
		handleLoadedTexture(moonTexture)
		textureFlag[0] = 1;
	}
	moonTexture.image.src = "/Public/image/moon.gif";

	crateTexture  = gl.createTexture();
	crateTexture.image = new Image();
	crateTexture.image.onload = function ()
	{
		handleLoadedTexture(crateTexture)
		textureFlag[1] = 1;
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
	shaderProgram.samplerUniform = 
		gl.getUniformLocation(shaderProgram, "uSampler");

	shaderProgram.materialAmbientColorUniform =
		gl.getUniformLocation(shaderProgram, "uMaterialAmbientColor");
	shaderProgram.materialDiffuseColorUniform =
		gl.getUniformLocation(shaderProgram, "uMaterialDiffuseColor");
	shaderProgram.materialSpecularColorUniform =
		gl.getUniformLocation(shaderProgram, "uMaterialSpecularColor");
	shaderProgram.materialShininessUniform =
		gl.getUniformLocation(shaderProgram, "uMaterialShininess");
	shaderProgram.materialEmissiveColorUniform =
		gl.getUniformLocation(shaderProgram, "uMaterialEmissiveColor");
	shaderProgram.showSpecularHighlightsUniform =
		gl.getUniformLocation(shaderProgram, "uShowSpecularHighlights");
	shaderProgram.useTexturesUniform =
		gl.getUniformLocation(shaderProgram, "uUseTextures");
	shaderProgram.ambientLightingColorUniform =
		gl.getUniformLocation(shaderProgram, "uAmbientLightingColor");
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

var laptopVertexPositionBuffer;
var laptopVertexNormalBuffer;
var laptopVertexTextureCoordBuffer;
var laptopVertexIndexBuffer;
function loadLaptop() 
{
	$.getJSON(
		"/Public/json/macbook.json", 
		function(data)
		{
			handleLoadedLaptop(data);
			textureFlag[2] = 1;
		}
	);
}
function handleLoadedLaptop(laptopData) 
{
	laptopVertexNormalBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, laptopVertexNormalBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(laptopData.vertexNormals), gl.STATIC_DRAW);
	laptopVertexNormalBuffer.itemSize = 3;
	laptopVertexNormalBuffer.numItems = laptopData.vertexNormals.length / 3;

	laptopVertexTextureCoordBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, laptopVertexTextureCoordBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(laptopData.vertexTextureCoords), gl.STATIC_DRAW);
	laptopVertexTextureCoordBuffer.itemSize = 2;
	laptopVertexTextureCoordBuffer.numItems = 
	laptopData.vertexTextureCoords.length / 2;

	laptopVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, laptopVertexPositionBuffer);
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(laptopData.vertexPositions), gl.STATIC_DRAW);
	laptopVertexPositionBuffer.itemSize = 3;
	laptopVertexPositionBuffer.numItems = laptopData.vertexPositions.length / 3;

	laptopVertexIndexBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, laptopVertexIndexBuffer);
	gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, 
		new Uint16Array(laptopData.indices), gl.STREAM_DRAW);
	laptopVertexIndexBuffer.itemSize = 1;
	laptopVertexIndexBuffer.numItems = laptopData.indices.length;
}

var cubeVertexPositionBuffer;
var cubeVertexNormalBuffer;
var cubeVertexTextureCoordBuffer;
var cubeVertexIndexBuffer;

var moonVertexPositionBuffer;
var moonVertexNormalBuffer;
var moonVertexTextureCoordBuffer;
var moonVertexIndexBuffer;

var laptopScreenVertexPositionBuffer;
var laptopScreenVertexNormalBuffer;
var laptopScreenVertexTextureCoordBuffer;

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
	var radius = 1;
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

	laptopScreenVertexPositionBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, laptopScreenVertexPositionBuffer);
	vertices = [
		 0.580687, 0.659, 0.813106,
		-0.580687, 0.659, 0.813107,
		 0.580687, 0.472, 0.113121,
		-0.580687, 0.472, 0.113121,
		];
	gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(vertices), gl.STATIC_DRAW);
	laptopScreenVertexPositionBuffer.itemSize = 3;
	laptopScreenVertexPositionBuffer.numItems = 4;

	laptopScreenVertexNormalBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, laptopScreenVertexNormalBuffer);
	var vertexNormals = [
		 0.000000, -0.965926, 0.258819,
		 0.000000, -0.965926, 0.258819,
		 0.000000, -0.965926, 0.258819,
		 0.000000, -0.965926, 0.258819,
	];
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(vertexNormals), gl.STATIC_DRAW);
	laptopScreenVertexNormalBuffer.itemSize = 3;
	laptopScreenVertexNormalBuffer.numItems = 4;

	laptopScreenVertexTextureCoordBuffer = gl.createBuffer();
	gl.bindBuffer(gl.ARRAY_BUFFER, laptopScreenVertexTextureCoordBuffer);
	var textureCoords = [
		1.0, 1.0,
		0.0, 1.0,
		1.0, 0.0,
		0.0, 0.0,
	];
	gl.bufferData(gl.ARRAY_BUFFER, 
		new Float32Array(textureCoords), gl.STATIC_DRAW);
	laptopScreenVertexTextureCoordBuffer.itemSize = 2;
	laptopScreenVertexTextureCoordBuffer.numItems = 4;
}
var laptopScreenAspectRatio = 1.66;

var moonAngle = 180;
var cubeAngle = 0;
function drawSceneOnLaptopScreen()
{
	gl.viewport(0, 0, rttFramebuffer.width, rttFramebuffer.height);
	gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

	mat4.perspective(pMatrix, 45, laptopScreenAspectRatio, 0.1, 100.0);

	gl.uniform1i(shaderProgram.showSpecularHighlightsUniform, false);
	gl.uniform3f(shaderProgram.ambientLightingColorUniform, 0.2, 0.2, 0.2);
	gl.uniform3f(shaderProgram.pointLightingLocationUniform, 0, 0, -5);
	gl.uniform3f(shaderProgram.pointLightingDiffuseColorUniform, 0.8, 0.8, 0.8);

	gl.uniform1i(shaderProgram.showSpecularHighlightsUniform, false);
	gl.uniform1i(shaderProgram.useTexturesUniform, true);

	gl.uniform3f(shaderProgram.materialAmbientColorUniform, 1.0, 1.0, 1.0);
	gl.uniform3f(shaderProgram.materialDiffuseColorUniform, 1.0, 1.0, 1.0);
	gl.uniform3f(shaderProgram.materialSpecularColorUniform, 0.0, 0.0, 0.0);
	gl.uniform1f(shaderProgram.materialShininessUniform, 0);
	gl.uniform3f(shaderProgram.materialEmissiveColorUniform, 0.0, 0.0, 0.0);

	mat4.identity(mvMatrix);

	mat4.translate(mvMatrix, mvMatrix, [0, 0, -5]);
	mat4.rotate(mvMatrix, mvMatrix, degToRad(30), [1, 0, 0]);

	mvPushMatrix();
	mat4.rotate(mvMatrix, mvMatrix, degToRad(moonAngle), [0, 1, 0]);
	mat4.translate(mvMatrix, mvMatrix, [2, 0, 0]);
	gl.activeTexture(gl.TEXTURE0);
	gl.bindTexture(gl.TEXTURE_2D, moonTexture);
	gl.uniform1i(shaderProgram.samplerUniform, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, 
		moonVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, moonVertexTextureCoordBuffer);
	gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, 
		moonVertexTextureCoordBuffer.itemSize, gl.FLOAT, false, 0, 0);

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
	mat4.translate(mvMatrix, mvMatrix, [1.25, 0, 0]);
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

	gl.bindTexture(gl.TEXTURE_2D, rttTexture);
	gl.generateMipmap(gl.TEXTURE_2D);
	gl.bindTexture(gl.TEXTURE_2D, null);
}
var laptopAngle = 0;

function drawScene()
{
	gl.bindFramebuffer(gl.FRAMEBUFFER, rttFramebuffer);
	drawSceneOnLaptopScreen();
	gl.bindFramebuffer(gl.FRAMEBUFFER, null);

	gl.viewport(0, 0, gl.viewportWidth, gl.viewportHeight);
	gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);

	mat4.perspective(pMatrix, 45, 
		gl.viewportWidth / gl.viewportHeight, 0.1, 100.0);

	mat4.identity(mvMatrix);

	mvPushMatrix();

	mat4.translate(mvMatrix, mvMatrix, [0, -0.4, -2.2]);
	mat4.rotate(mvMatrix, mvMatrix, degToRad(laptopAngle), [0, 1, 0]);
	mat4.rotate(mvMatrix, mvMatrix, degToRad(-90), [1, 0, 0]);
	
	gl.uniform1i(shaderProgram.showSpecularHighlightsUniform, true);
	gl.uniform3f(shaderProgram.pointLightingLocationUniform, -1, 2, -1);
	gl.uniform3f(shaderProgram.ambientLightingColorUniform, -1, 2, -1);

	gl.uniform3f(shaderProgram.ambientLightingColorUniform, 0.2, 0.2, 0.2);
	gl.uniform3f(shaderProgram.pointLightingDiffuseColorUniform, 0.8, 0.8, 0.8);
	gl.uniform3f(shaderProgram.pointLightingSpecularColorUniform, 0.8, 0.8, 0.8);


	gl.uniform3f(shaderProgram.materialAmbientColorUniform, 1.0, 1.0, 1.0);
	gl.uniform3f(shaderProgram.materialDiffuseColorUniform, 1.0, 1.0, 1.0);
	gl.uniform3f(shaderProgram.materialSpecularColorUniform, 1.5, 1.5, 1.5);
	gl.uniform1f(shaderProgram.materialShininessUniform, 5);
	gl.uniform3f(shaderProgram.materialEmissiveColorUniform, 0.0, 0.0, 0.0);
	gl.uniform1i(shaderProgram.useTexturesUniform, false);

	if (laptopVertexPositionBuffer) 
	{
		gl.bindBuffer(gl.ARRAY_BUFFER, laptopVertexPositionBuffer);
		gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, 
			laptopVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

		gl.bindBuffer(gl.ARRAY_BUFFER, laptopVertexTextureCoordBuffer);
		gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, 
			laptopVertexTextureCoordBuffer.itemSize, gl.FLOAT, false, 0, 0);

		gl.bindBuffer(gl.ARRAY_BUFFER, laptopVertexNormalBuffer);
		gl.vertexAttribPointer(shaderProgram.vertexNormalAttribute, 
			laptopVertexNormalBuffer.itemSize, gl.FLOAT, false, 0, 0);

		gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, laptopVertexIndexBuffer);
		setMatrixUniforms();
		gl.drawElements(gl.TRIANGLES, 
			laptopVertexIndexBuffer.numItems, gl.UNSIGNED_SHORT, 0);
	}

	gl.uniform3f(shaderProgram.materialAmbientColorUniform, 0.0, 0.0, 0.0);
	gl.uniform3f(shaderProgram.materialDiffuseColorUniform, 0.0, 0.0, 0.0);
	gl.uniform3f(shaderProgram.materialSpecularColorUniform, 0.5, 0.5, 0.5);
	gl.uniform1f(shaderProgram.materialShininessUniform, 20);
	gl.uniform3f(shaderProgram.materialEmissiveColorUniform, 1.5, 1.5, 1.5);
	gl.uniform1i(shaderProgram.useTexturesUniform, true);

	gl.bindBuffer(gl.ARRAY_BUFFER, laptopScreenVertexPositionBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexPositionAttribute, 
		laptopScreenVertexPositionBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, laptopScreenVertexNormalBuffer);
	gl.vertexAttribPointer(shaderProgram.vertexNormalAttribute, 
		laptopScreenVertexNormalBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.bindBuffer(gl.ARRAY_BUFFER, laptopScreenVertexTextureCoordBuffer);
	gl.vertexAttribPointer(shaderProgram.textureCoordAttribute, 
		laptopScreenVertexTextureCoordBuffer.itemSize, gl.FLOAT, false, 0, 0);

	gl.activeTexture(gl.TEXTURE0);
	gl.bindTexture(gl.TEXTURE_2D, rttTexture);
	gl.uniform1i(shaderProgram.samplerUniform, 0);

	setMatrixUniforms();
	gl.drawArrays(gl.TRIANGLE_STRIP, 0, 
		laptopScreenVertexPositionBuffer.numItems);

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

		laptopAngle -= 0.005 * elapsed;
	}
	lastTime = timeNow;
}
</script>
```