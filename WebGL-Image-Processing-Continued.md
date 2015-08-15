# WebGL 图像处理延续部分 #

这片文章是 [WebGL 图像处理](http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html)部分的延续内容.如果你还没有阅读它，建议最好阅读一下,[可以从这里点击开始阅读](http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html).

下一个关于图像处理的显著问题就是如何应用多重效果?读者当然可以尝试着写一下着色器.生成一个 UI 来让用户使用不同的着色器选择他们希望的效果.这通常是不太可能的，因为这个技术通常需要[实时的渲染效果](http://www.youtube.com/watch?v=cQUn0Zeh-0Q).

一种比较灵活的方式是使用两种或更多的纹理和渲染效果来交替渲染,每次应用一个效果，然后反复应用.  


    Original Image -> [Blur]-> Texture 1
    Texture 1  -> [Sharpen] -> Texture 2
    Texture 2  -> [Edge Detect] -> Texture 1
    Texture 1  -> [Blur]-> Texture 2
    Texture 2  -> [Normal]  -> Canvas


要做到这一点，就需要创建帧缓存区.在 WebGL 和 OpenGL 中，帧缓存区实际上是一个非常不正式的名称. WebGL/OpenGL 中的帧缓存实际上仅仅是一些状态的集合，而不是真正的缓存.但是，每当一种纹理到达帧缓存，我们就会渲染出这种纹理.

首先让我们把[旧的纹理创建代码](http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html)写成一个函数.  

     function createAndSetupTexture(gl) {
    var texture = gl.createTexture();
    gl.bindTexture(gl.TEXTURE_2D, texture);
     
    // Set up texture so we can render any size image and so we are
    // working with pixels.
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.CLAMP_TO_EDGE);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.NEAREST);
    gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.NEAREST);
     
    return texture;
      }
     
      // Create a texture and put the image in it.
      var originalImageTexture = createAndSetupTexture(gl);
      gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);   

然后，我们使用这两个函数来生成两种问题，并且附在两个帧缓存中.

    // create 2 textures and attach them to framebuffers.
      var textures = [];
      var framebuffers = [];
      for (var ii = 0; ii < 2; ++ii) {
    var texture = createAndSetupTexture(gl);
    textures.push(texture);
     
    // make the texture the same size as the image
    gl.texImage2D(
    gl.TEXTURE_2D, 0, gl.RGBA, image.width, image.height, 0,
    gl.RGBA, gl.UNSIGNED_BYTE, null);
     
    // Create a framebuffer
    var fbo = gl.createFramebuffer();
    framebuffers.push(fbo);
    gl.bindFramebuffer(gl.FRAMEBUFFER, fbo);
     
    // Attach a texture to it.
    gl.framebufferTexture2D(
    gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D, texture, 0);
      }   

现在，我们生成一些核的集合，然后存储到列表里来应用.  

      // Define several convolution kernels
      var kernels = {
    normal: [
      0, 0, 0,
      0, 1, 0,
      0, 0, 0
    ],
    gaussianBlur: [
      0.045, 0.122, 0.045,
      0.122, 0.332, 0.122,
      0.045, 0.122, 0.045
    ],
    unsharpen: [
      -1, -1, -1,
      -1,  9, -1,
      -1, -1, -1
    ],
    emboss: [
       -2, -1,  0,
       -1,  1,  1,
    0,  1,  2
    ]
      };
     
      // List of effects to apply.
      var effectsToApply = [
    "gaussianBlur",
    "emboss",
    "gaussianBlur",
    "unsharpen"
      ];   

最后，我们应用每一个，然后交替渲染.

    // start with the original image
      gl.bindTexture(gl.TEXTURE_2D, originalImageTexture);
     
      // don't y flip images while drawing to the textures
      gl.uniform1f(flipYLocation, 1);
     
      // loop through each effect we want to apply.
      for (var ii = 0; ii < effectsToApply.length; ++ii) {
    // Setup to draw into one of the framebuffers.
    setFramebuffer(framebuffers[ii % 2], image.width, image.height);
     
    drawWithKernel(effectsToApply[ii]);
     
    // for the next draw, use the texture we just rendered to.
    gl.bindTexture(gl.TEXTURE_2D, textures[ii % 2]);
      }
     
      // finally draw the result to the canvas.
      gl.uniform1f(flipYLocation, -1);  // need to y flip for canvas
      setFramebuffer(null, canvas.width, canvas.height);
      drawWithKernel("normal");
     
      function setFramebuffer(fbo, width, height) {
    // make this the framebuffer we are rendering to.
    gl.bindFramebuffer(gl.FRAMEBUFFER, fbo);
     
    // Tell the shader the resolution of the framebuffer.
    gl.uniform2f(resolutionLocation, width, height);
     
    // Tell webgl the viewport setting needed for framebuffer.
    gl.viewport(0, 0, width, height);
      }
     
      function drawWithKernel(name) {
    // set the kernel
    gl.uniform1fv(kernelLocation, kernels[name]);
     
    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 6);
      }   


下面是更灵活的 UI 的可交互版本.勾选相应的效果即可检查效果.

#####################################################
######################################
[
点击这里打开另一个窗口来显示示例](http://webglfundamentals.org/webgl/webgl-2d-image-processing.html)   


有些事情，我们要做完整.  

以空值调用 `gl.bindFramebuffer` 即可告诉 WebGL 程序员希望渲染到画板而不是帧缓存中的纹理.  

WebGL 不得不将投[影矩阵](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)转换为像素.这是基于 `gl.viewport` 的设置.当我们初始化 WebGL的时候， `gl.viewport` 的设置默认为画板的尺寸.因为，我们会将帧缓存渲染为不同的尺寸，所以画板需要设置合适的视图.   

最后，在[原始例子](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)中，当需要渲染的时候，我们会翻转 Y 坐标.这是因为 WebGL 会以 0 来显示面板. 0 表示是左侧底部的坐标，这不同于 2D 图像的顶部左侧的坐标.当渲染为帧缓存时就不需要了.这是因为帧缓存并不会显示出来.其部分是顶部还是底部是无关紧要的.所有重要的就是像素0，0在帧缓存里就对应着0.为了解决这一问题,我们可以通过是否在着色器中添加更多输入信息的方法来设置是否快读交替.   

    <script id="2d-vertex-shader" type="x-shader/x-vertex">
    ...
    uniform float u_flipY;
    ...
     
    void main() {
       ...
       gl_Position = vec4(clipSpace * vec2(1, u_flipY), 0, 1);
       ...
    }
    </script>


当我们渲染的时候，就可以设置它.

    
      var flipYLocation = gl.getUniformLocation(program, "u_flipY");
      ...
      // don't flip
      gl.uniform1f(flipYLocation, 1);
      ...
      // flip
      gl.uniform1f(flipYLocation, -1);


在这个简单的例子中，通过使用单个GLSL程序可以实现多个效果.
如果你想做完整的图像处理你可能需要许多GLSL程序.一个程序实现色相、饱和度和亮度调整。另一个实现亮度和对比度。一个实现反相,另一个用于调整水平.你可能需要更改代码以更新GLSL程序和更新特定程序的参数.我本来考虑写出这个例子,但这是一个练习，所以最好留给读者自己实现,因为多个 GLSL 项目中每一种方法都有自己的参数，可能意味着需要一些重大重构，这很可能导致成为意大利面条似的大混乱.

我希望这和前面的示例中可以看出 WebGL 似乎更平易近人,我希望从 2D 方面入手，以有助于使 WebGL 更容易理解.   

如果有时间的话，我会尽量写一些关于如何做 3D 的文章以及更多关于 WebGL 是做什么的细节.下一步我们将学习如何使用两种或更多的纹理.