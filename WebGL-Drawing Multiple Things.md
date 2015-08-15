#WebGL - 绘制多个东西

这篇文章是<a href="http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html">之前 WebGL 文章</a>的延续。如果你还没有读过那些文章，我建议你先读完那些文章。  

在 WebGL 中第一次得到东西后最常见的问题之一是，我怎样绘制多个东西。  

除了少数例外情况，首先要意识到的东西是，WebGL 就像某人写的包含某个函数，而不是向函数中传递大量参数，相反，你有一个独自的函数来绘制东西，同时有 70 + 函数来为一个函数设置状态。因此，例如假设你有一个可以绘制一个圆的函数。你可以像如下一样编写程序  


    function drawCircle(centerX, centerY, radius, color) { ... }

或者你可以像如下一样编写代码  

    var centerX;
    var centerY;
    var radius;
    var color;
     
    function setCenter(x, y) {
       centerX = x;
       centerY = y;
    }
     
    function setRadius(r) {
       radius = r;
    }
     
    function setColor(c) {
       color = c;
    }
     
    function drawCircle() {
       ...
    }

WebGL 以第二种方式工作。函数，诸如 `gl.createBuffer`, `gl.bufferData`, `gl.createTexture` 和 `gl.texImage2D`，让你可以上传缓冲区（ 顶点 ）和质地 （ 颜色，等等 ）数据到 **WebGL**。`gl.createProgram`, `gl.createShader`, `gl.compileProgram` 和 `gl.linkProgram` 让你可以创建你的 **GLSL** 着色器。当 `gl.drawArrays `或者 `gl.drawElements` 函数被调用时，几乎所有的 **WebGL** 的其余函数都正在设置要被使用的全局变量或者状态。  

我们知道，这个典型的 WebGL 程序基本上遵循这个结构。  

在初始化时  

- 创建所有的着色器和程序  

- 创建缓冲区和上传顶点数据 

- 创建质地和上传质地数据

在渲染时

- 清除并且设置视区和其他全局状态（启用深度测试，开启扑杀，等等）

- 对于你想要绘制的每一件事

 	- 为你想要书写的程序调用 `gl.useProgram`
	
	
	- 为你想要绘制的东西设置属性  
	 
		- 对于每个属性调用 `gl.bindBuffer`, `gl.vertexAttribPointer`, `gl.enableVertexAttribArray` 函数       
		
	- 为你想要绘制的东西设置制服
	
		- 为每一个制服调用 `gl.uniformXXX`  
		
		- 调用 `gl.activeTexture` 和 `gl.bindTexture` 来为质地单元分配质地  
		 
	- 调用 `gl.drawArrays` 或者 `gl.drawElements`  
	  
这就是最基本的。怎样组织你的代码来完成这一任务取决于你。  


一些事情诸如上传质地数据（ 甚至顶点数据 ）可能会异步的发生，因为你需要等待他们在网上下载完。  

让我们来做一个简单的应用程序来绘制 3 种东西。一个立方体，一个球体和一个圆锥体。  

我不会去详谈如何计算立方体，球体和圆锥体的数据。假设我们有函数来创建它们，然后我们返回<a href="http://webglfundamentals.org/webgl/lessons/webgl-less-code-more-fun.html">在之前篇章中介绍的 bufferInfo 对象</a>。  

所以这里是代码。我们的着色器，与从我们的<a href="http://webglfundamentals.org/webgl/lessons/webgl-3d-perspective.html">角度看示例</a>的一个简单着色器相同，除了我们已经通过添加另外一个 `u-colorMult` 来增加顶点颜色。 

    // Passed in from the vertex shader.
    varying vec4 v_color;
     
    uniform vec4 u_colorMult;
     
    void main() {
       gl_FragColor = v_color * u_colorMult;
    }

在初始化时

    // Our uniforms for each thing we want to draw
    var sphereUniforms = {
      u_colorMult: [0.5, 1, 0.5, 1],
      u_matrix: makeIdentity(),
    };
    var cubeUniforms = {
      u_colorMult: [1, 0.5, 0.5, 1],
      u_matrix: makeIdentity(),
    };
    var coneUniforms = {
      u_colorMult: [0.5, 0.5, 1, 1],
      u_matrix: makeIdentity(),
    };
     
    // The translation for each object.
    var sphereTranslation = [  0, 0, 0];
    var cubeTranslation   = [-40, 0, 0];
    var coneTranslation   = [ 40, 0, 0];

在绘制时  

    var sphereXRotation =  time;
    var sphereYRotation =  time;
    var cubeXRotation   = -time;
    var cubeYRotation   =  time;
    var coneXRotation   =  time;
    var coneYRotation   = -time;
     
    // ------ Draw the sphere --------
     
    gl.useProgram(programInfo.program);
     
    // Setup all the needed attributes.
    setBuffersAndAttributes(gl, programInfo.attribSetters, sphereBufferInfo);
     
    sphereUniforms.u_matrix = computeMatrix(
    viewMatrix,
    projectionMatrix,
    sphereTranslation,
    sphereXRotation,
    sphereYRotation);
     
    // Set the uniforms we just computed
    setUniforms(programInfo.uniformSetters, sphereUniforms);
     
    gl.drawArrays(gl.TRIANGLES, 0, sphereBufferInfo.numElements);
     
    // ------ Draw the cube --------
     
    // Setup all the needed attributes.
    setBuffersAndAttributes(gl, programInfo.attribSetters, cubeBufferInfo);
     
    cubeUniforms.u_matrix = computeMatrix(
    viewMatrix,
    projectionMatrix,
    cubeTranslation,
    cubeXRotation,
    cubeYRotation);
     
    // Set the uniforms we just computed
    setUniforms(programInfo.uniformSetters, cubeUniforms);
     
    gl.drawArrays(gl.TRIANGLES, 0, cubeBufferInfo.numElements);
     
    // ------ Draw the cone --------
     
    // Setup all the needed attributes.
    setBuffersAndAttributes(gl, programInfo.attribSetters, coneBufferInfo);
     
    coneUniforms.u_matrix = computeMatrix(
    viewMatrix,
    projectionMatrix,
    coneTranslation,
    coneXRotation,
    coneYRotation);
     
    // Set the uniforms we just computed
    setUniforms(programInfo.uniformSetters, coneUniforms);
     
    gl.drawArrays(gl.TRIANGLES, 0, coneBufferInfo.numElements);

如下所示  

![](/images/webgl-drawing-multiple-things1.png)

<a href="http://webglfundamentals.org/webgl/webgl-multiple-objects-manual.html">单击这里将会打开一个单独的窗体</a>

需要注意的一件事情是，因为我们只有一个着色器程序，我们仅调用了 `gl.useProgram` 一次。如果我们有不同的着色器程序，你需要在使用每个程序之前调用  `gl.useProgram`。  

这是另外一个值得去简化的地方。这里结合了 3 个主要的有效的事情。  

1. 一个着色器程序（同时还有它的制服和属性 信息/设置）

2. 你想要绘制的东西的缓冲区和属性

3. 制服需要用给出的着色器来绘制你想要绘制的东西

所以，一个简单的简化可能会绘制出一个数组的东西，同时在这个数组中将 3 个东西放在一起。  

    var objectsToDraw = [
      {
    programInfo: programInfo,
    bufferInfo: sphereBufferInfo,
    uniforms: sphereUniforms,
      },
      {
    programInfo: programInfo,
    bufferInfo: cubeBufferInfo,
    uniforms: cubeUniforms,
      },
      {
    programInfo: programInfo,
    bufferInfo: coneBufferInfo,
    uniforms: coneUniforms,
      },
    ];

在绘制时，我们仍然需要更新矩阵  

    var sphereXRotation =  time;
    var sphereYRotation =  time;
    var cubeXRotation   = -time;
    var cubeYRotation   =  time;
    var coneXRotation   =  time;
    var coneYRotation   = -time;
     
    // Compute the matrices for each object.
    sphereUniforms.u_matrix = computeMatrix(
    viewMatrix,
    projectionMatrix,
    sphereTranslation,
    sphereXRotation,
    sphereYRotation);
     
    cubeUniforms.u_matrix = computeMatrix(
    viewMatrix,
    projectionMatrix,
    cubeTranslation,
    cubeXRotation,
    cubeYRotation);
     
    coneUniforms.u_matrix = computeMatrix(
    viewMatrix,
    projectionMatrix,
    coneTranslation,
    coneXRotation,
    coneYRotation);

但是这个绘制代码现在只是一个简单的循环  

    // ------ Draw the objects --------
     
    objectsToDraw.forEach(function(object) {
      var programInfo = object.programInfo;
      var bufferInfo = object.bufferInfo;
     
      gl.useProgram(programInfo.program);
     
      // Setup all the needed attributes.
      setBuffersAndAttributes(gl, programInfo.attribSetters, bufferInfo);
     
      // Set the uniforms.
      setUniforms(programInfo.uniformSetters, object.uniforms);
     
      // Draw
      gl.drawArrays(gl.TRIANGLES, 0, bufferInfo.numElements);
    });

这可以说是大多数 3 D 引擎的主渲染循环都存在的。一些代码所在的地方或者是代码决定将什么放入 `objectsToDraw` 的列表中，基本上是这样。  

![](/images/webgl-drawing-multiple-things2.png)
<a href="http://webglfundamentals.org/webgl/webgl-multiple-objects-list.html">单击这里将出现一个单独的窗体</a> 


这里有几个基本的优化。如果这个我们想要绘制东西的程序与我们已经绘制东西的之前的程序一样，就不需要重新调用 `gl.useProgram` 了。同样，如果我们现在正在绘制的与我们之前已经绘制的东西有相同的形状 / 几何 / 顶点，就不需要再次设置上面的东西了。  

所以，一个很简单的优化会与以下代码类似   

    var lastUsedProgramInfo = null;
    var lastUsedBufferInfo = null;
     
    objectsToDraw.forEach(function(object) {
      var programInfo = object.programInfo;
      var bufferInfo = object.bufferInfo;
      var bindBuffers = false;
     
      if (programInfo !== lastUsedProgramInfo) {
    lastUsedProgramInfo = programInfo;
    gl.useProgram(programInfo.program);
     
    // We have to rebind buffers when changing programs because we
    // only bind buffers the program uses. So if 2 programs use the same
    // bufferInfo but the 1st one uses only positions the when the
    // we switch to the 2nd one some of the attributes will not be on.
    bindBuffers = true;
      }
     
      // Setup all the needed attributes.
      if (bindBuffers || bufferInfo != lastUsedBufferInfo) {
    lastUsedBufferInfo = bufferInfo;
    setBuffersAndAttributes(gl, programInfo.attribSetters, bufferInfo);
      }
     
      // Set the uniforms.
      setUniforms(programInfo.uniformSetters, object.uniforms);
     
      // Draw
      gl.drawArrays(gl.TRIANGLES, 0, bufferInfo.numElements);
    });
    
这次让我们来绘制更多的对象。与之前的仅仅 3 个东西不同，让我们做一系列的东西来绘制更大的东西。  

    // put the shapes in an array so it's easy to pick them at random
    var shapes = [
      sphereBufferInfo,
      cubeBufferInfo,
      coneBufferInfo,
    ];
     
    // make 2 lists of objects, one of stuff to draw, one to manipulate.
    var objectsToDraw = [];
    var objects = [];
     
    // Uniforms for each object.
    var numObjects = 200;
    for (var ii = 0; ii < numObjects; ++ii) {
      // pick a shape
      var bufferInfo = shapes[rand(0, shapes.length) | 0];
     
      // make an object.
      var object = {
    uniforms: {
      u_colorMult: [rand(0, 1), rand(0, 1), rand(0, 1), 1],
      u_matrix: makeIdentity(),
    },
    translation: [rand(-100, 100), rand(-100, 100), rand(-150, -50)],
    xRotationSpeed: rand(0.8, 1.2),
    yRotationSpeed: rand(0.8, 1.2),
      };
      objects.push(object);
     
      // Add it to the list of things to draw.
      objectsToDraw.push({
    programInfo: programInfo,
    bufferInfo: bufferInfo,
    uniforms: object.uniforms,
      });
    }

在渲染时  

    // Compute the matrices for each object.
    objects.forEach(function(object) {
      object.uniforms.u_matrix = computeMatrix(
      viewMatrix,
      projectionMatrix,
      object.translation,
      object.xRotationSpeed * time,
      object.yRotationSpeed * time);
    });
	
然后使用上面的循环绘制对象  

![](/images/webgl-drawing-multiple-things3.png)
<a href="http://webglfundamentals.org/webgl/webgl-multiple-objects-list-optimized.html">单击这里会出现一个单独的窗体</a>

你也可以通过 `programInfo` 和 / 或者 `bufferInfo` 来对列表进行排序，以便优化开始的更加频繁。大多数游戏引擎都是这样做。不幸的是它不是那么简单。如果你现在正在绘制的任何东西都不透明，然后你可以只排序。但是，一旦你需要绘制半透明的东西，你就需要以特定的顺序来绘制它们。大多数 3 D 引擎都通过有 2 个或者更多的要绘制的对象的列表来处理这个问题。不透明的东西有一个列表。透明的东西有另外一个列表。不透明的列表按程序和几何来排序。透明的列表按深度排序。对于其他东西，诸如覆盖或后期处理效果，还会有其他单独的列表。  

<a href="http://webglfundamentals.org/webgl/webgl-multiple-objects-list-optimized-sorted.html">这里有一个已经排好序的例子</a>。在我的机器上，我得到了未排序的 ~31 fps 和排好序的 ~37.发现几乎增长了 20 %。但是，它是在最糟糕的案例和最好的案例相比较下，大多数的程序将会做的更多，因此，它可能对于所有情况来说不值得考虑，但是最特别的案例值得考虑。  

注意到你不可能仅仅使用任何着色器来仅仅绘制任何几何是非常重要的。例如，一个需要法线的着色器在没有法线的几何情况下将不会起作用。同样，一个组要质地的着色器在没有质地时将不会工作。   

选择一个像 <a href="http://threejs.org/">Three.js</a> 的 3D 库是很重要的，这是众多原因之一，因为它会为你处理所有这些东西。你创建了一些几何，你告诉 three.js 你想让它怎样呈现，它会在运行时产生着色器来处理你需要的东西。几乎所有的 3D 引擎都将它们从 Unity3D 到虚幻的 Crytek 源。一些离线就可以生成它们，但是最重要的事是意识到是它们生成了着色器。  

当然，你正在读这些文章的原因，是你想要知道接下来将会发生什么。你自己写任何东西都是非常好且有趣的。意识到 <a href="http://webglfundamentals.org/webgl/lessons/webgl-2d-vs-3d-library.html">WebGL 是超级低水平的</a>是非常重要的，因此如果你想要自己做，这里有许多你可以做的工作，这经常包括写一个着色器生成器，因为不同的功能往往需要不同的着色器。  

你将会注意到我并没有在循环中放置 `computeMatrix`。那是因为呈现应该与计算矩阵分开。从<a href="http://webglfundamentals.org/webgl/lessons/webgl-scene-graph.html">场景图和我们将另一篇文章中读到的内容</a>，计算矩阵是非常常见的。  

现在，我们已经有了一个绘制多对象的框架，<a href="http://webglfundamentals.org/webgl/lessons/webgl-text-html.html">让我们来绘制一些文本</a>。









