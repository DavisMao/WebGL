# WebGL 是如何工作的 #

这部分是上一节[WebGL 基础](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)的延续.在继续之前，我们需要讨论 WebGL 和 GPU 是如何运作的. GPU 有两个基础任务，第一个就是将点处理为投影矩阵.第二部分就是基于第一部分将相应的像素点描绘出来.   

当用户调用   

    gl.drawArrays(gl.TRIANGLE, 0, 9);   

这里的9就意味着“处理 9 个顶点”，所以就有 9 个顶点需要被处理.   

![](http://webglfundamentals.org/webgl/lessons/resources/vertex-shader-anim.gif)   

上图左侧的是用户自己提供的数据. 定点着色器就是用户在 GLSL 中写的函数. 处理每个定点时，均会被调用一次. 用户可以将投影矩阵的值存储在特定的变量 `gl_Position`  中. GPU会处理这些值，并将他们存储在其内部.  

假设用户希望绘制三角形 `TRIANGLES`, 那么每次绘制时，上述的第一部分就会产生三个顶点，然后GPU会使用他们来绘制三角形. 首先 GPU 会将三个顶点对应的像素绘制出来，然后将三角形光栅化，或者说是使用像素点绘制出来. 对每一个像素点，GPU 都会调用用户定义的片段着色器来确定该像素点该涂成什么颜色. 当然，用户定义的片段着色器必须在 `gl_FragColor`  变量中设置对应的值.   

我们例子中的片段着色器中并没有存储每一个像素的信息. 我们可以在其中存储更丰富的信息. 我们可以为每一个值定义不同的意义从定点着色器传递到片段着色器.  

作为一个简单的例子，我们将直接计算出来的投影矩阵坐标从定点着色器传递给片段着色器.   

我们将绘制一个简单的三角形. 我们在[上个例子](http://webglfundamentals.org/webgl/lessons/webgl-2d-matrices.html)的基础上更改一下.   


    // Fill the buffer with the values that define a triangle.
    function setGeometry(gl) {
      gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array([
     0, -100,
       150,  125,
      -175,  100]),
      gl.STATIC_DRAW);
    }   

然后，我们绘制三个顶点.   

    // Draw the scene.
    function drawScene() {
      ...
      // Draw the geometry.
      gl.drawArrays(gl.TRIANGLES, 0, 3);
    }   


然后，我们可以在顶点着色器中定义变量来将数据传递给片段着色器.  

    varying vec4 v_color;
    ...
    void main() {
      // Multiply the position by the matrix.
      gl_Position = vec4((u_matrix * vec3(a_position, 1)).xy, 0, 1);
     
      // Convert from clipspace to colorspace.
      // Clipspace goes -1.0 to +1.0
      // Colorspace goes from 0.0 to 1.0
      v_color = gl_Position * 0.5 + 0.5;
    }

然后，我们在片段着色器中声明相同的变量.  


    precision mediump float;
     
    varying vec4 v_color;
     
    void main() {
      gl_FragColor = v_color;
    }   


WebGL 将会连接顶点着色器中的变量和片段着色器中的相同名字和类型的变量.  

下面是可以交互的版本.   

##############################################
##############################################

[点击这里可以新的窗口打开示例](http://webglfundamentals.org/webgl/webgl-2d-triangle-with-position-for-color.html)   

移动、缩放或旋转这个三角形. 注意由于颜色是从投影矩阵计算而来，所以，颜色并不会随着三角形的移动而一直一样.他们完全是根据背景色设定的.   

现在我们考虑下面的内容. 我们仅仅计算三个顶点. 我们的顶点着色器被调用了三次，因此，仅仅计算了三个颜色.而我们的三角形可以有好多颜色，这就是为何被称为 *`varying`*.   

WebGL 使用了我们为每个定点计算的三个值，然后将三角形光栅化. 对于每一个像素，都会使用被修改过的值来调用片段着色器.  

基于上述例子，我们以三个顶点开始.  

![](http://i.imgur.com/3XcNPQC.png)   

我们的顶点着色器会引用矩阵来转换、旋转、缩放和转化为投影矩阵.转换、旋转和缩放的默认值是转换为200,150，旋转为0，缩放为1,1，所以实际上只进行转换. 我们的后台缓存是400x300. 我们的顶点矩阵应用矩阵然后计算下面的三个投影矩阵顶点.  

![](http://i.imgur.com/tjAxxoe.png) 
  

同样也会将这些转换到颜色空间上，然后将他们写到我们声明的多变变量 v_color.   


![](http://i.imgur.com/OFgAThJ.png)   

这三个值会写回到 v_color，然后它会被传递到片段着色器用于每一个像素进行着色.  

##########################################################
#####################################################

v_color 被修改为 v0，v1和v2三个值中的一个.  

我们也可以在顶点着色器中存储更多的数据以便往片段着色器中传递.  所以，对于以两种颜色绘制包含两个三角色的矩形的例子.为了实现这个例子，我们需要往顶点着色器中附加更多的属性，以便传递更多的数据，这些数据会直接传递到片段着色器中.   

    attribute vec2 a_position;
    attribute vec4 a_color;
    ...
    varying vec4 v_color;
     
    void main() {
       ...
      // Copy the color from the attribute to the varying.
      v_color = a_color;
    }    

我们现在需要使用 WebGL 颜色相关的功能.   

	`// look up where the vertex data needs to go.
 		 var positionLocation = gl.getAttribLocation	(program, "a_position");
 	 var colorLocation = gl.getAttribLocation(program, "a_color");
 	 ...
 	 // Create a buffer for the colors.
 	 var buffer = gl.createBuffer();
 	 gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
 	 gl.enableVertexAttribArray(colorLocation);
 	 gl.vertexAttribPointer(colorLocation, 4, gl.FLOAT, false, 0, 0);

  	// Set the colors.
  		setColors(gl);
 
	// Fill the buffer with colors for the 2 triangles
	// that make the rectangle.
	function setColors(gl) {
 	 // Pick 2 random colors.
 	 var r1 = Math.random();
 	 var b1 = Math.random();
 	 var g1 = Math.random();
	  var r2 = Math.random();
	  var b2 = Math.random();
	  var g2 = Math.random();
 	 gl.bufferData(
 	 gl.ARRAY_BUFFER,
 	 new Float32Array(
	[ r1, b1, g1, 1,
  	r1, b1, g1, 1,
 	 r1, b1, g1, 1,
 	 r2, b2, g2, 1,
 	 r2, b2, g2, 1,
 	 r2, b2, g2, 1]),
 	 gl.STATIC_DRAW);
	}  `   


下面是结果.    




##################################################
##################################################

[点击这里可以新的窗口打开示例](http://webglfundamentals.org/webgl/webgl-2d-rectangle-with-2-colors.html)    


注意，在上面的例子中，有两个苦点颜色的三角形.我们仍将要传递的值存储在多变变量中，所以，该变量会相关三角形区域内改变.我们只是对于每个三角形的三个顶点使用了相同的颜色.如果我们使用了不同的颜色，我们可以看到整个渲染过程.   

    / Fill the buffer with colors for the 2 triangles
    // that make the rectangle.
    function setColors(gl) {
      // Make every vertex a different color.
      gl.bufferData(
      gl.ARRAY_BUFFER,
      new Float32Array(
    [ Math.random(), Math.random(), Math.random(), 1,
      Math.random(), Math.random(), Math.random(), 1,
      Math.random(), Math.random(), Math.random(), 1,
      Math.random(), Math.random(), Math.random(), 1,
      Math.random(), Math.random(), Math.random(), 1,
      Math.random(), Math.random(), Math.random(), 1]),
      gl.STATIC_DRAW);
    }    


现在我们看一下被渲染后的多变变量.  

####################################################
####################################################

[点击这里可以新的窗口打开示例](http://webglfundamentals.org/webgl/webgl-2d-rectangle-with-random-colors.html)   


从定点着色器往片段着色器可以传递更多更丰富的数据.如果我们来检验[图像处理示例](http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html)，就会发现在纹理坐标中会传递更多的属性.   

## 缓存和属性指令究竟做了什么？ ##   

缓存是获取定点和顶点相关数据到GPU中的方法.`gl.createBuffer` 用于创建缓存. `gl.bindBuffer` 方法用于将缓存激活来处于准备工作的状态. `gl.bufferData` 方法可以将数据拷贝到缓存中.    

一旦，数据到了缓存中，就需要告诉 WebGL 如何从里面错去数据，并将它提供给顶点着色器以给相应的属性赋值.   

为了实现这个功能，首先我们需要求 WebGL 提供一个属性存储位置. 下面是示例代码.   

    // look up where the vertex data needs to go.
    var positionLocation = gl.getAttribLocation(program, "a_position");
    var colorLocation = gl.getAttribLocation(program, "a_color");    

一旦我们知道了对应的属性，我们可以触发两个指令.   

    gl.enableVertexAttribArray(location);    

这个指令会告诉 WebGL 我们希望将缓存中的数据赋值给一个变量.   

    gl.vertexAttribPointer(
    location,
    numComponents,
    typeOfData,
    normalizeFlag,
    strideToNextPieceOfData,
    offsetIntoBuffer);    

这个指令会告诉 WebGL 会从缓存中获取数据，这个缓存会与 `gl.bindBuffer` 绑定. 每个顶点可以有1到4个部件，数据的类型可以是BYTE,FLOAT,INT,UNSIGNED_SHORT等. 跳跃意味着从数据的这片到那片会跨越多少个字节.跨越多远会以偏移量的方式存储在缓存中.   

部件的数目一般会是1到4.  

如果每个数据类型仅使用一个缓存，那么跨越和偏移量都会是 0.跨越为 0 意味着“使用一个跨越连匹配类型和尺寸”.偏移量为 0 意味着是在缓存的开头部分. 将这个值赋值为除 O 之外其他的值会实现更为灵活的功能. 虽然在性能方面它有些优势，但是并不值得搞得很复杂，除非程序员希望将 WebGL 运用到极致.   

我希望到此就可以将缓冲区和属性相关内容已经介绍清楚了. 

下面我们学习[着色器和 GLSL](http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html).    

## 什么是顶点属性指针 vertexAttribPointer 的规范化标志 normalizeFlag ? ## 

规范化标志应用于非浮点指针类型. 如果该值置为 false, 就意味着该值就会被翻译为类型. BYTE 的标示范围是-128到127.UNSIGNED_BYTE 范围是0到255，SHORT 是从-32768到32767.   

如果将规范化标志置为 true，那么BYTE的标示范围将为变为-1.0到+1.0,UNSIGNED_BYTE 将会变为0.0到+1.0，规范化后的 SHORT 将会变为 -1.0 到+1.0，它将有比 BYTE 更高的精确度.  

标准化数据最通用的地方就是用于颜色. 大部分时候，颜色范围为0.0到1.0. 红色、绿色和蓝色需要个浮点型的值来表示，alpha 需要 16 字节来表示顶点的每个颜色. 如果要实现更为复杂的图形，可以增加更多的字节. 相反的，程序员可以将颜色转为 UNSIGNED_BYTE 类型，这个类型使用0表示0.0，使用255表示1.0. 那么仅需要4个字节来表示顶点的每个颜色，这将节省 75% 的存储空间.   


我们按照下面的方式来更改我们的代码. 当我们告诉 WebGL 如何获取颜色.   


     gl.vertexAttribPointer(colorLocation, 4, gl.UNSIGNED_BYTE, true, 0, 0);   


我们可以使用下面的代码来填充我们的缓冲区.


    // Fill the buffer with colors for the 2 triangles
    // that make the rectangle.
    function setColors(gl) {
      // Pick 2 random colors.
      var r1 = Math.random() * 256; // 0 to 255.99999
      var b1 = Math.random() * 256; // these values
      var g1 = Math.random() * 256; // will be truncated
      var r2 = Math.random() * 256; // when stored in the
      var b2 = Math.random() * 256; // Uint8Array
      var g2 = Math.random() * 256;
     
      gl.bufferData(
      gl.ARRAY_BUFFER,
      new Uint8Array(   // Uint8Array
    [ r1, b1, g1, 255,
      r1, b1, g1, 255,
      r1, b1, g1, 255,
      r2, b2, g2, 255,
      r2, b2, g2, 255,
      r2, b2, g2, 255]),
      gl.STATIC_DRAW);
    }


下面是个例子.

####################################################################

[点击这里可以新的窗口打开示例](http://webglfundamentals.org/webgl/webgl-2d-rectangle-with-2-byte-colors.html) 


  



