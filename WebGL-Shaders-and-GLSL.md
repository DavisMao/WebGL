# WebGL 着色器和 GLSL #

这部分是 [WebGL 基础](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)的延续. 如果你不想阅读 WebGL 如何工作，你也许会希望[查看这个](http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html).   

我们已经讨论了一些关于着色器和 GLSL 的相关内容，但是仍然介绍的不是很仔细.可以通过一个例子来更清楚的了解的更清楚.  

正如 [WebGL 是如何工作](http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html)的中讲到的，每次绘制，都需要两个着色器, 分别是顶点着色器和片段着色器. 每个着色器都是一个函数. 顶点着色器和片段着色器都是链接在程序中的.一个典型的 WebGL 程序都会不包含很多这样的着色器程序.   

## 顶点着色器 ##

顶点着色器的任务就是产生投影矩阵的坐标. 其形式如下：

    void main() {
       gl_Position = doMathToMakeClipspaceCoordinates
    }     

程序员实现的着色器对每个定点都会被调用一次.每次调用程序员都需要设置特定的全局变量 `gl_Position` 来表示一些投影矩阵的坐标.   

顶点着色器需要数据，它以下面三种方式来获取这些数据. 

1. 属性（从缓冲区中获取数据）
1. 一致变量（每次绘画调用时都保持一致的值）
1. 纹理（从像素中得到的数据）   

**属性**

最常用的方式就是使用缓存区和属性. [WebGL 如何工作](http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html)中已经涵盖了缓冲区和属性.    

程序员可以以下面的方式创建缓存区.   

    var buf = gl.createBuffer();    

在这些缓存中存储数据.   

    gl.bindBuffer(gl.ARRAY_BUFFER, buf);
    gl.bufferData(gl.ARRAY_BUFFER, someData, gl.STATIC_DRAW);    


于是，给定一个着色器程序，程序员可以去查找属性的位置. 


    var positionLoc = gl.getAttribLocation(someShaderProgram, "a_position");    


下面告诉 WebGL 如何从缓存区中获取数据并存储到属性中.  

    // turn on getting data out of a buffer for this attribute
    gl.enableVertexAttribArray(positionLoc);
     
    var numComponents = 3;  // (x, y, z)
    var type = gl.FLOAT;
    var normalize = false;  // leave the values as they are
    var offset = 0; // start at the beginning of the buffer
    var stride = 0; // how many bytes to move to the next vertex
    // 0 = use the correct stride for type and numComponents
     
    gl.vertexAttribPointer(positionLoc, numComponents, type, false, stride, offset);    


在 [WebGL 基础](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)中我们展示了我们可以在着色器中附加一些逻辑，然后将值直接传递.    


    attribute vec4 a_position;
     
    void main() {
       gl_Position = a_position;
    }     


如果我们可以将投影矩阵放入我们的缓存区中，它就会开始运作.   

属性可以使用 float，vec2，vec3，vec4，mat2，mat3和mat4作为类型.  

**一致性变量**   

对于顶点着色器，一致性变量就是在绘画每次调用时，在着色器中一直保持不变的值. 下面是一个往顶点中添加偏移量着色器的例子.  

    attribute vec4 a_position;
    uniform vec4 u_offset;
     
    void main() {
       gl_Position = a_position + u_offset;
    }   

下面，我们需要对每一个顶点都偏移一定量. 首先，我们需要先找到一致变量的位置.   

    var offsetLoc = gl.getUniformLocation(someProgram, "u_offset");   

然后，我们在绘制前需要设置一致性变量.   

    gl.uniform4fv(offsetLoc, [1, 0, 0, 0]);  // offset it to the right half the screen   

一致性变量可以有很多种类型. 对每一种类型都可以调用相应的函数来设置.   


    gl.uniform1f (floatUniformLoc, v); // for float
    gl.uniform1fv(floatUniformLoc, [v]);   // for float or float array
    gl.uniform2f (vec2UniformLoc,  v0, v1);// for vec2
    gl.uniform2fv(vec2UniformLoc,  [v0, v1]);  // for vec2 or vec2 array
    gl.uniform3f (vec3UniformLoc,  v0, v1, v2);// for vec3
    gl.uniform3fv(vec3UniformLoc,  [v0, v1, v2]);  // for vec3 or vec3 array
    gl.uniform4f (vec4UniformLoc,  v0, v1, v2, v4);// for vec4
    gl.uniform4fv(vec4UniformLoc,  [v0, v1, v2, v4]);  // for vec4 or vec4 array
     
    gl.uniformMatrix2fv(mat2UniformLoc, false, [  4x element array ])  // for mat2 or mat2 array
    gl.uniformMatrix3fv(mat3UniformLoc, false, [  9x element array ])  // for mat3 or mat3 array
    gl.uniformMatrix4fv(mat4UniformLoc, false, [ 17x element array ])  // for mat4 or mat4 array
     
    gl.uniform1i (intUniformLoc,   v); // for int
    gl.uniform1iv(intUniformLoc, [v]); // for int or int array
    gl.uniform2i (ivec2UniformLoc, v0, v1);// for ivec2
    gl.uniform2iv(ivec2UniformLoc, [v0, v1]);  // for ivec2 or ivec2 array
    gl.uniform3i (ivec3UniformLoc, v0, v1, v2);// for ivec3
    gl.uniform3iv(ivec3UniformLoc, [v0, v1, v2]);  // for ivec3 or ivec3 array
    gl.uniform4i (ivec4UniformLoc, v0, v1, v2, v4);// for ivec4
    gl.uniform4iv(ivec4UniformLoc, [v0, v1, v2, v4]);  // for ivec4 or ivec4 array
     
    gl.uniform1i (sampler2DUniformLoc,   v);   // for sampler2D (textures)
    gl.uniform1iv(sampler2DUniformLoc, [v]);   // for sampler2D or sampler2D array
     
    gl.uniform1i (samplerCubeUniformLoc,   v); // for samplerCube (textures)
    gl.uniform1iv(samplerCubeUniformLoc, [v]); // for samplerCube or samplerCube array

 

一般类型都有 bool，bvec2，bvec3 和 bvec4. 他们相应的调用函数形式为 gl.uniform?f? 或 gl.uniform?i?.   

可以一次性设置数组中的所有一致性变量. 比如:

    // in shader
    uniform vec2 u_someVec2[3];
     
    // in JavaScript at init time
    var someVec2Loc = gl.getUniformLocation(someProgram, "u_someVec2");
     
    // at render time
    gl.uniform2fv(someVec2Loc, [1, 2, 3, 4, 5, 6]);  // set the entire array of u_someVec3


 
但是，如果程序员希望单独设置数组中的成员，那么必须单个的查询每个成员的位置.  

    // in JavaScript at init time
    var someVec2Element0Loc = gl.getUniformLocation(someProgram, "u_someVec2[0]");
    var someVec2Element1Loc = gl.getUniformLocation(someProgram, "u_someVec2[1]");
    var someVec2Element2Loc = gl.getUniformLocation(someProgram, "u_someVec2[2]");
     
    // at render time
    gl.uniform2fv(someVec2Element0Loc, [1, 2]);  // set element 0
    gl.uniform2fv(someVec2Element1Loc, [3, 4]);  // set element 1
    gl.uniform2fv(someVec2Element2Loc, [5, 6]);  // set element 2



类似的，可以创建一个结构体.   

    struct SomeStruct {
      bool active;
      vec2 someVec2;
    };
    uniform SomeStruct u_someThing;   

程序员可以单独的查询每一个成员.

    var someThingActiveLoc = gl.getUniformLocation(someProgram, "u_someThing.active");
    var someThingSomeVec2Loc = gl.getUniformLocation(someProgram, "u_someThing.someVec2");


**顶点着色器中的纹理**   

参考[片段着色器中的纹理](http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#textures-in-fragment-shaders)   

## 片段着色器 ##

片段着色器的任务就是为当前被栅格化的像素提供颜色.它通常以下面的方式呈现出来.   

    precision mediump float;
     
    void main() {
       gl_FragColor = doMathToMakeAColor;
    }   

片段着色器对每一个像素都会调用一次.每次调用都会设置全局变量 `gl_FragColor` 来设置一些颜色.   

片段着色器需要存储获取数据，通常有下面这三种方式.

1. 一致变量（每次绘制像素点时都会调用且一直保持一致）
1. 纹理（从像素中获取数据）
1. 多变变量（从定点着色器中传递出来且被栅格化的值）    


**片段着色器中的一致变量**

参考[顶点着色器中的一致变量](http://webglfundamentals.org/webgl/lessons/webgl-shaders-and-glsl.html#uniforms). 


**片段着色器中的纹理**   

我们可以从纹理中获取值来创建 `sampler2D` 一致变量, 然后使用 GLSL 函数 `texture2D` 来从中获取值.  

    precision mediump float;
     
    uniform sampler2D u_texture;
     
    void main() {
       vec2 texcoord = vec2(0.5, 0.5)  // get a value from the middle of the texture
       gl_FragColor = texture2D(u_texture, texcoord);
    }


从纹理中提取的值是要[依据很多设置](http://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html)的.最基本的，我们需要创建并在文理中存储值.比如，

    var tex = gl.createTexture();
    gl.bindTexture(gl.TEXTURE_2D, tex);
    var level = 0;
    var width = 2;
    var height = 1;
    var data = new Uint8Array([255, 0, 0, 255, 0, 255, 0, 255]);
    gl.texImage2D(gl.TEXTURE_2D, level, gl.RGBA, width, height, 0, gl.RGBA, gl.UNSIGNED_BYTE, data);    


然后，在着色器程序中查询一致变量的位置.  

    var someSamplerLoc = gl.getUniformLocation(someProgram, "u_texture");


WebGL 需要将它绑定到纹理单元中.   

    var unit = 5;  // Pick some texture unit
    gl.activeTexture(gl.TEXTURE0 + unit);
    gl.bindTexture(gl.TEXTURE_2D, tex);   


然后告知着色器哪个单元会被绑定到纹理中.   

    gl.uniform1i(someSamplerLoc, unit);   


**多变变量**

多变变量是从顶点着色器往片段着色器中传递的值,这些在[WebGL 如何工作的](http://webglfundamentals.org/webgl/lessons/webgl-how-it-works.html)已经涵盖了.   

为了使用多变变量，需要在顶点着色器和片段着色器中均设置匹配的多变变量.我们在顶点着色器中设置多变变量。当 WebGL 绘制像素时，它会栅格化该值，然后传递到片段着色器中相对应的片段着色器.  

## 顶点着色器 ##

    attribute vec4 a_position;
     
    uniform vec4 u_offset;
     
    varying vec4 v_positionWithOffset;
     
    void main() {
      gl_Position = a_position + u_offset;
      v_positionWithOffset = a_position + u_offset;
    }


片段着色器


    precision mediump float;
     
    varying vec4 v_positionWithOffset;
     
    void main() {
      // convert from clipsapce (-1 <-> +1) to color space (0 -> 1).
      vec4 color = v_positionWithOffset * 0.5 + 0.5
      gl_FragColor = color;
    }


上面的例子是无关紧要的例子.它一般不会直接复制投影矩阵的值到片段着色器中.

**GLSL**  

GLSL是图像库着色器语言的简称. 语言着色器就是被写在这里.它具有一些 JavaScript 中不存在的独特的特性. 它用于实现一些逻辑来渲染图像.比如，它可以创建类似于 vec2，vec3和vec4分别表示2、3、4个值.类似的，mat2，mat3 和 mat4 来表示2x2,3x3,4x4的矩阵.可以实现 vec 来乘以一个标量.  

    vec4 a = vec4(1, 2, 3, 4);
    vec4 b = a * 2.0;
    // b is now vec4(2, 4, 6, 8);

类似的，可以实现矩阵的乘法和矩阵的向量乘法.

    mat4 a = ???
    mat4 b = ???
    mat4 c = a * b;
     
    vec4 v = ???
    vec4 y = c * v;


也可以选择 vec的部分，比如，vec4

    vec4 v;

- v.x 等价于 v.s，v.r，v[0]
- v.y 等价于 v.t，v.g，v[1]
- v.z 等价于 v.p，v.b，v[2]
- v.w 等价于 v.q，v.a，v[3]

可以调整 vec 组件意味着可以交换或重复组件.   


    v.yyyy

这等价于

    vec4(v.y, v.y, v.y, v.y)

类似的
    
    v.bgra

等价于

    vec4(v.b, v.g, v.r, v.a)

当创建一个 vec 或 一个 mat时，程序员可以一次操作多个部分.比如，

    vec4(v.rgb, 1)

这等价于

    vec4(v.r, v.g, v.b, 1)

你可能意识到 GLSL 是一种很严格类型的语言.  

    float f = 1;  // ERROR 1 is an int. You can't assign an int to a float   

正确的方式如下：

    float f = 1.0;  // use float
    float f = float(1)  // cast the integer to a float


上面例子的 vec4(v.rgb, 1) 并不会对 1 进行混淆，这是因为 vec4 是类似于 float（1）.

GLSL 是内置函数的分支.可以同时操作多个组件.比如，
    
    T sin(T angle)

这意味着 T 可以是 float，vec2，vec3 或 vec4.如果用户在vec4中传递数据。也就是说v是vec4，

    vec4 s = sin(v);

折等价于

    vec4 s = vec4(sin(v.x), sin(v.y), sin(v.z), sin(v.w));


有时候，一个参数是float，另一个是 T. 这意味着 float 将应用到所有的部件.比如，如果v1，v2是vec4，f是flat，然后

    vec4 m = mix(v1, v2, f);

这等价于

    vec4 m = vec4(
      mix(v1.x, v2.x, f),
      mix(v1.y, v2.y, f),
      mix(v1.z, v2.z, f),
      mix(v1.w, v2.w, f));

可以在 [WebGL 参考目录](https://www.khronos.org/files/webgl/webgl-reference-card-1_0.pdf)的最后一页查看 GLSL 函数. 如果你希望查看一些仔细的可以查看 [GLSL 规范](https://www.khronos.org/files/opengles_shading_language.pdf).   

## 混合起来看 ##

这是整个系列的文章. WebGL 是关于创建各种着色器的,将数据存储在这些着色器上，然后调用 gl.drawArrays 或 gl.drawElements 来使 WebGL 处理顶点通过为每个顶点调用当前的顶点着色器，然后为每一个像素调用当前的片段着色器.   

实际上，着色器的创建需要写几行代码.因为，这些代码在大部分 WebGL 程序中的是一样的，又因为一旦写了，可以几乎可以忽略他们如何编译GLSL着色器和链接成一个着色器程序。

此外，读者这里有两个方向可以发展，一是如果读者对图像处理比较感兴趣，那么可以阅读 [2D 图像处理技术](http://webglfundamentals.org/webgl/lessons/webgl-image-processing.html).  如果读者对转换、旋转和缩放感兴趣，可以阅读[这个材料](http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html).  

    






