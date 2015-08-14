#  WebGL 基础 #
WebGL的出现使得在浏览器上面实时显示3D图像变为了可能，但是几乎所有人都不知道WebGL本质上是基于光栅化的 API ,而不是基于 3D 的 API .  


这里我们需要深入解释一下.   


WebGL 只关注两个方面，即投影矩阵的坐标和投影矩阵的颜色.使用 WebGL 的程序员的任务就是实现具有投影矩阵坐标和颜色的WebGL对象即可。程序员可以使用“着色器”来完成上述任务.定点着色器可以提供投影矩阵的坐标，片段着色器可以提供投影矩阵的颜色。  

无论程序员要实现的图形尺寸有多大， 其投影矩阵的坐标的范围始终是从 -1 到 1.下面是一个关于实现 WebGL 对象的一个简单例子.   

    // Get A WebGL context
    var canvas = document.getElementById("canvas");
    var gl = canvas.getContext("experimental-webgl");
     
    // setup a GLSL program
    var program = createProgramFromScripts(gl, ["2d-vertex-shader", "2d-fragment-shader"]);
    gl.useProgram(program);
     
    // look up where the vertex data needs to go.
    var positionLocation = gl.getAttribLocation(program, "a_position");
     
    // Create a buffer and put a single clipspace rectangle in
    // it (2 triangles)
    var buffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
    gl.bufferData(
    gl.ARRAY_BUFFER,
    new Float32Array([
    -1.0, -1.0,
     1.0, -1.0,
    -1.0,  1.0,
    -1.0,  1.0,
     1.0, -1.0,
     1.0,  1.0]),
    gl.STATIC_DRAW);
    gl.enableVertexAttribArray(positionLocation);
    gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0);
     
    // draw
    gl.drawArrays(gl.TRIANGLES, 0, 6);   


下面是两个着色器.  

    <script id="2d-vertex-shader" type="x-shader/x-vertex">
    attribute vec2 a_position;
     
    void main() {
      gl_Position = vec4(a_position, 0, 1);
    }
    </script>
     
    <script id="2d-fragment-shader" type="x-shader/x-fragment">
    void main() {
      gl_FragColor = vec4(0, 1, 0, 1);  // green
    }
    </script>    

它将绘出一个绿色的长方形来填充整个画板.   


![](http://i.imgur.com/7CPaFdH.png)
[点击这里打开另一个窗口来显示](http://webglfundamentals.org/webgl/webgl-fundamentals.html)
后面内容还会更精彩，我们继续：-P

我们再次降调一下，无论画板尺寸多大，投影矩阵坐标的范围只会在 -1 到 1 之间.从上面的例子中，我们可以看出我们只是将位置信息直接写在了程序里. 因为位置信息已经在投影矩阵中，所以并没有其他额外的工作要做. 如果程序员想实现 3D 的效果，那么程序员可以使用着色器来将 3D 转换为投影矩阵，这是因为 *WebGL* 是基于光栅的 API.   

对于 2D 的图像，程序员也许会使用像素而不是投影矩阵来表述尺寸，那么这里我们就更改这里的着色器，使得我们实现的矩形可以以像素的方式来度量，下面是新的顶点着色器.


    <script id="2d-vertex-shader" type="x-shader/x-vertex">
    attribute vec2 a_position;
     
    uniform vec2 u_resolution;
     
    void main() {
       // convert the rectangle from pixels to 0.0 to 1.0
       vec2 zeroToOne = a_position / u_resolution;
     
       // convert from 0->1 to 0->2
       vec2 zeroToTwo = zeroToOne * 2.0;
     
       // convert from 0->2 to -1->+1 (clipspace)
       vec2 clipSpace = zeroToTwo - 1.0;
     
       gl_Position = vec4(clipSpace, 0, 1);
    }
    </script>    

下面我们将我们的数据从投影矩阵改为像素.

    // set the resolution
    var resolutionLocation = gl.getUniformLocation(program, "u_resolution");
    gl.uniform2f(resolutionLocation, canvas.width, canvas.height);
     
    // setup a rectangle from 10,20 to 80,30 in pixels
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
    10, 20,
    80, 20,
    10, 30,
    10, 30,
    80, 20,
    80, 30]), gl.STATIC_DRAW);    


![](http://i.imgur.com/2drHv88.png)

[点击这里打开另一个窗口来显示](http://webglfundamentals.org/webgl/webgl-2d-rectangle.html)     

下面我们将上述关于矩阵的实现写成函数以便可以以函数调用的方式来实现不同尺寸的矩阵. 然而，这里的颜色应该是可变的.   

首先，我们为片段着色器设计一个关于颜色的输入.   

    <script id="2d-fragment-shader" type="x-shader/x-fragment">
    precision mediump float;
     
    uniform vec4 u_color;
     
    void main() {
       gl_FragColor = u_color;
    }
    </script>    

下面是实现绘画50个尺寸和颜色均随机的矩阵的代码.    

    var colorLocation = gl.getUniformLocation(program, "u_color");
      ...
      // Create a buffer
      var buffer = gl.createBuffer();
      gl.bindBuffer(gl.ARRAY_BUFFER, buffer);
      gl.enableVertexAttribArray(positionLocation);
      gl.vertexAttribPointer(positionLocation, 2, gl.FLOAT, false, 0, 0);
     
      // draw 50 random rectangles in random colors
      for (var ii = 0; ii < 50; ++ii) {
    // Setup a random rectangle
    setRectangle(
    gl, randomInt(300), randomInt(300), randomInt(300), randomInt(300));
     
    // Set a random color.
    gl.uniform4f(colorLocation, Math.random(), Math.random(), Math.random(), 1);
     
    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 6);
      }
    }
     
    // Returns a random integer from 0 to range - 1.
    function randomInt(range) {
      return Math.floor(Math.random() * range);
    }
     
    // Fills the buffer with the values that define a rectangle.
    function setRectangle(gl, x, y, width, height) {
      var x1 = x;
      var x2 = x + width;
      var y1 = y;
      var y2 = y + height;
      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
     x1, y1,
     x2, y1,
     x1, y2,
     x1, y2,
     x2, y1,
     x2, y2]), gl.STATIC_DRAW);
    }    


下面就是实现出来的效果.

![](http://i.imgur.com/3Mwhw40.png)   

[点击这里打开另一个窗口来显示](http://webglfundamentals.org/webgl/webgl-2d-rectangles.html)    


笔者这里希望读者到此可以看出 WebGL 实质上是一种轻量级的 API.  但是，它可以实现较为复杂的 3D 效果，其复杂性由程序员定制. WebGL API 本身是 2D 的且相对比较简单.  

如果读者完全不了解 WebGL，并且完全不了解 GLSL、着色器、GPU，那么建议阅读 [WebGL 是如何工作的](http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html).   

此外，读者这里有两个方向可以发展，一是如果读者对图像处理比较感兴趣，那么可以阅读 [2D 图像处理技术](http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html).  如果读者对转换、旋转和缩放感兴趣，可以阅读[这个材料](http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html).   

type="x-shader/x-vertex" 和 type="x-shader/x-fragment" 意味着什么？   

`<script>` 标签意味着其中包含了 JavaScript 代码. 你也可以不使用 type 或者使用 `type="javascript"` 、 `type="text/javascript"`，这样浏览器就会将其中的内容解析为 JavaScript脚本.如果用户将一些其他一些内容放进去，那么浏览器就会忽略掉 script 脚本里的内容.   

用户可以使用这个特点在 script 脚本里面存储着色器. 甚至，我们可以在这个脚本里实现将着色器编译为定点或片段着色器.  

在这种情况下， `createProgramFromScripts` 可以以特定的编号来搜索脚本，然后寻找 `type` 来决定要创建的着色器是什么类型。   

`createProgramFromScripts` 是[代码示例](http://webglfundamentals.org/webgl/lessons/webgl-boilerplate.html)中的一部分，也是几乎每个 WebGL 程序都需要的。
