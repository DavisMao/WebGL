# WebGL 图像处理 #

在 WebGL 中图像处理是很简单的.多么简单？阅读下面的内容.这是 [WebGL 基础](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)的延续.如果你还没有阅读过，建议阅读这[些资料](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html).    

为了在 WebGL 中绘制图像，我们需要使用纹理.类似于当渲染代替像素时，WebGL会需要操作投影矩阵的坐标,WebGL 读取纹理时需要获取纹理坐标.纹理坐标范围是从 0.0 到 1.0.   

因为我们仅需要绘制由两个三角形组成的矩形，我们需要告诉 WebGL 在矩阵中纹理对应的那个点.我们可以使用特殊的被称为多变变量会将这些信息从顶点着色器传递到片段着色器.WebGL 将会插入这些值，这些值会在顶点着色器中，当对每个像素绘制时均会调用片段着色器.  


通过[以往关于顶点着色器的文章](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)，我们需要在纹理坐标传递过程中添加更多的信息，然后将他们传递到片段着色器中.   

    attribute vec2 a_texCoord;
    ...
    varying vec2 v_texCoord;
     
    void main() {
       ...
       // pass the texCoord to the fragment shader
       // The GPU will interpolate this value between points
       v_texCoord = a_texCoord;
    }
    

然后,我们提供一个片段着色器来查找颜色纹理。

    <script id="2d-fragment-shader" type="x-shader/x-fragment">
    precision mediump float;
     
    // our texture
    uniform sampler2D u_image;
     
    // the texCoords passed in from the vertex shader.
    varying vec2 v_texCoord;
     
    void main() {
       // Look up a color from the texture.
       gl_FragColor = texture2D(u_image, v_texCoord);
    }
    </script>


最后，我们需要加载一个图片，然后创建一个问题，将该图片传递到纹理里面.因为，是在浏览器里面显示，所以图片是异步加载，所以我们安置我们的代码来等待纹理的加载.一旦，加载完成就可以绘制.

    function main() {
      var image = new Image();
      image.src = "http://someimage/on/our/server";  // MUST BE SAME DOMAIN!!!
      image.onload = function() {
    render(image);
      }
    }
     
    function render(image) {
      ...
      // all the code we had before.
      ...
      // look up where the texture coordinates need to go.
      var texCoordLocation = gl.getAttribLocation(program, "a_texCoord");
     
      // provide texture coordinates for the rectangle.
      var texCoordBuffer = gl.createBuffer();
      gl.bindBuffer(gl.ARRAY_BUFFER, texCoordBuffer);
      gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
      0.0,  0.0,
      1.0,  0.0,
      0.0,  1.0,
      0.0,  1.0,
      1.0,  0.0,
      1.0,  1.0]), gl.STATIC_DRAW);
      gl.enableVertexAttribArray(texCoordLocation);
      gl.vertexAttribPointer(texCoordLocation, 2, gl.FLOAT, false, 0, 0);
     
      // Create a texture.
      var texture = gl.createTexture();
      gl.bindTexture(gl.TEXTURE_2D, texture);
     
      // Set the parameters so we can render any size image.
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
      gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
     
      // Upload the image into the texture.
      gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);
      ...
    }

如下是 WebGL 渲染出来的图像.

![](http://i.imgur.com/G6OfQqJ.png)
[
点击这里在另一个窗口打开示例
](http://webglfundamentals.org/webgl/webgl-2d-image.html)


下面我们对这个图片进行一些操作，来交换图片中的红色和蓝色.

    ...
    gl_FragColor = texture2D(u_image, v_texCoord).bgra;
    ...


现在红色和蓝色已经被交换了.效果如下:

![](http://i.imgur.com/jm636Vk.png)
[
点击这里在另一个窗口打开示例
](http://webglfundamentals.org/webgl/webgl-2d-image-red2blue.html)


假如我们想做一些图像处理，那么我们可以看一下其他像素.自从 WebGL 引用纹理的纹理坐标从0.0到1.0.然后，我们可以计算移动的多少个像素 onePixel = 1.0 / textureSize.

这里有个片段着色器来平均纹理中每个像素的左侧和右侧的像素.  

    <script id="2d-fragment-shader" type="x-shader/x-fragment">
    precision mediump float;
     
    // our texture
    uniform sampler2D u_image;
    uniform vec2 u_textureSize;
     
    // the texCoords passed in from the vertex shader.
    varying vec2 v_texCoord;
     
    void main() {
       // compute 1 pixel in texture coordinates.
       vec2 onePixel = vec2(1.0, 1.0) / u_textureSize;
     
       // average the left, middle, and right pixels.
       gl_FragColor = (
       texture2D(u_image, v_texCoord) +
       texture2D(u_image, v_texCoord + vec2(onePixel.x, 0.0)) +
       texture2D(u_image, v_texCoord + vec2(-onePixel.x, 0.0))) / 3.0;
    }
    </script>


然后,我们需要通过JavaScript传递出纹理的大小.

    ...
    var textureSizeLocation = gl.getUniformLocation(program, "u_textureSize");
    ...
    // set the size of the image
    gl.uniform2f(textureSizeLocation, image.width, image.height);
    ...


比较上述两个图片

![](http://i.imgur.com/20X0ox5.png)

[点击这里在另一个窗口打开示例](http://webglfundamentals.org/webgl/webgl-2d-image-blend.html)

现在,我们知道如何让我们使用引用其他像素卷积内核做一些常见的图像处理.这里，我们会使用3x3的内核.卷积内核就是一个3x3的矩阵，矩阵中的每个条目代表有多少像素渲染.然后，我们将这个结果除以内核的权重或1.0.[这里是一个非常好的参考文章](http://docs.gimp.org/en/plug-in-convmatrix.html).[这里有另一篇文章显示出一些实际代码，它是使用c++写的.](http://www.codeproject.com/KB/graphics/ImageConvolution.aspx)


在我们的例子中我们要在着色器中做这样工作，这里是一个新的片段着色器。

    <script id="2d-fragment-shader" type="x-shader/x-fragment">
    precision mediump float;
     
    // our texture
    uniform sampler2D u_image;
    uniform vec2 u_textureSize;
    uniform float u_kernel[9];
    uniform float u_kernelWeight;
     
    // the texCoords passed in from the vertex shader.
    varying vec2 v_texCoord;
     
    void main() {
       vec2 onePixel = vec2(1.0, 1.0) / u_textureSize;
       vec4 colorSum =
     texture2D(u_image, v_texCoord + onePixel * vec2(-1, -1)) * u_kernel[0] +
     texture2D(u_image, v_texCoord + onePixel * vec2( 0, -1)) * u_kernel[1] +
     texture2D(u_image, v_texCoord + onePixel * vec2( 1, -1)) * u_kernel[2] +
     texture2D(u_image, v_texCoord + onePixel * vec2(-1,  0)) * u_kernel[3] +
     texture2D(u_image, v_texCoord + onePixel * vec2( 0,  0)) * u_kernel[4] +
     texture2D(u_image, v_texCoord + onePixel * vec2( 1,  0)) * u_kernel[5] +
     texture2D(u_image, v_texCoord + onePixel * vec2(-1,  1)) * u_kernel[6] +
     texture2D(u_image, v_texCoord + onePixel * vec2( 0,  1)) * u_kernel[7] +
     texture2D(u_image, v_texCoord + onePixel * vec2( 1,  1)) * u_kernel[8] ;
     
       // Divide the sum by the weight but just use rgb
       // we'll set alpha to 1.0
       gl_FragColor = vec4((colorSum / u_kernelWeight).rgb, 1.0);
    }
    </script>


在 JavaScript 中，我们需要提供一个卷积内核和它的权重.

     function computeKernelWeight(kernel) {
       var weight = kernel.reduce(function(prev, curr) {
       return prev + curr;
       });
       return weight <= 0 ? 1 : weight;
     }
     
     ...
     var kernelLocation = gl.getUniformLocation(program, "u_kernel[0]");
     var kernelWeightLocation = gl.getUniformLocation(program, "u_kernelWeight");
     ...
     var edgeDetectKernel = [
     -1, -1, -1,
     -1,  8, -1,
     -1, -1, -1
     ];
     gl.uniform1fv(kernelLocation, edgeDetectKernel);
     gl.uniform1f(kernelWeightLocation, computeKernelWeight(edgeDetectKernel));
     ...

我们在列表框内选择不同的内核.

###########################################################################################################

[点击这里在另一个窗口打开示例](http://webglfundamentals.org/webgl/webgl-2d-image-3x3-convolution.html)


我们希望这里的文章讲解的 WebGL中的图像处理是比较简单的.下面，我们将讲解[如何在一个图像上应用更多的效果](http://webglfundamentals.org/webgl/lessons/webgl-image-processing-continued.html).


'u_image'并没设置，它是如何工作的？一致性变量的默认值是0，所以u_image默认值只纹理单元0.纹理单元0也是默认激活的纹理,所以调用 bindTexture 会绑定纹理到纹理单元0上.WebGL 会有一个纹理单元数组.纹理单元设置方法是通过查询样本一致变量的位置，然后设置希望引用的纹理单元的索引.比如：

    var textureUnitIndex = 6; // use texture unit 6.
    var u_imageLoc = gl.getUniformLocation(
    program, "u_image");
    gl.uniform1i(u_imageLoc, textureUnitIndex);

为了不同的单元设置纹理，可以调用 gl.activeTexture ，然后绑定该单元到纹理中.比如.

    // Bind someTexture to texture unit 6.
    gl.activeTexture(gl.TEXTURE6);
    gl.bindTexture(gl.TEXTURE_2D, someTexture);

如下也可以起作用

    var textureUnitIndex = 6; // use texture unit 6.
    // Bind someTexture to texture unit 6.
    gl.activeTexture(gl.TEXTURE0 + textureUnitIndex);
    gl.bindTexture(gl.TEXTURE_2D, someTexture);


## GLSL 变量中的前缀a_,u_,v_ 是什么意思？ ##

这仅仅是一种卷积的命名方法.他们不是必需的,但这使得它更容易看到一眼的值来自哪里.属性修饰符a_ 意味着他来自缓冲区.u_是用于给着色器输入参数的一致性变量.v_ 指的是多变变量用于顶点着色器给片段着色器传递数据.详细内容[参考这里](http://webglfundamentals.org/webgl/lessons/webgl-how-to-works.html). 