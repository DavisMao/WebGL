#WebGL - 更少的代码，更多的乐趣  

这篇文章是关于 WebGL 的一系列文章的续篇。首先<a href="http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html">以基本元素开始</a>。如果你没有读过那些，请先查看它们。  

WebGL 程序要求你编写必须编译和链接的着色器程序，然后你需要查看对于这些着色器程序的输入的位置。这些输入被称为制服和属性，同时用来查找它们的位置的代码可能是冗长而乏味的。  

假设我们已经有了<a href="http://webglfundamentals.org/webgl/lessons/webgl-boilerplate.html">用来编译和链接着色器程序的 WebGL 代码的典型样本</a>。下面给出了一组着色器。  

顶点着色器：  

    uniform mat4 u_worldViewProjection;
    uniform vec3 u_lightWorldPos;
    uniform mat4 u_world;
    uniform mat4 u_viewInverse;
    uniform mat4 u_worldInverseTranspose;
    
    attribute vec4 a_position;
    attribute vec3 a_normal;
    attribute vec2 a_texcoord;
    
    varying vec4 v_position;
    varying vec2 v_texCoord;
    varying vec3 v_normal;
    varying vec3 v_surfaceToLight;
    varying vec3 v_surfaceToView;
    
    void main() {
      v_texCoord = a_texcoord;
      v_position = (u_worldViewProjection * a_position);
      v_normal = (u_worldInverseTranspose * vec4(a_normal, 0)).xyz;
      v_surfaceToLight = u_lightWorldPos - (u_world * a_position).xyz;
      v_surfaceToView = (u_viewInverse[3] - (u_world * a_position)).xyz;
      gl_Position = v_position;
    }

片段着色器：

    precision mediump float;
    
    varying vec4 v_position;
    varying vec2 v_texCoord;
    varying vec3 v_normal;
    varying vec3 v_surfaceToLight;
    varying vec3 v_surfaceToView;
    
    uniform vec4 u_lightColor;
    uniform vec4 u_ambient;
    uniform sampler2D u_diffuse;
    uniform vec4 u_specular;
    uniform float u_shininess;
    uniform float u_specularFactor;
    
    vec4 lit(float l ,float h, float m) {
      return vec4(1.0,
      max(l, 0.0),
      (l > 0.0) ? pow(max(0.0, h), m) : 0.0,
      1.0);
    }
    
    void main() {
      vec4 diffuseColor = texture2D(u_diffuse, v_texCoord);
      vec3 a_normal = normalize(v_normal);
      vec3 surfaceToLight = normalize(v_surfaceToLight);
      vec3 surfaceToView = normalize(v_surfaceToView);
      vec3 halfVector = normalize(surfaceToLight + surfaceToView);
      vec4 litR = lit(dot(a_normal, surfaceToLight),
    dot(a_normal, halfVector), u_shininess);
      vec4 outColor = vec4((
      u_lightColor * (diffuseColor * litR.y + diffuseColor * u_ambient +
    u_specular * litR.z * u_specularFactor)).rgb,
      diffuseColor.a);
      gl_FragColor = outColor;
    }

你最终不得不像以下这样编写代码，来对所有要绘制的各种各样的值进行查找和设置。  

    // At initialization time
    var u_worldViewProjectionLoc   = gl.getUniformLocation(program, "u_worldViewProjection");
    var u_lightWorldPosLoc = gl.getUniformLocation(program, "u_lightWorldPos");
    var u_worldLoc = gl.getUniformLocation(program, "u_world");
    var u_viewInverseLoc   = gl.getUniformLocation(program, "u_viewInverse");
    var u_worldInverseTransposeLoc = gl.getUniformLocation(program, "u_worldInverseTranspose");
    var u_lightColorLoc= gl.getUniformLocation(program, "u_lightColor");
    var u_ambientLoc   = gl.getUniformLocation(program, "u_ambient");
    var u_diffuseLoc   = gl.getUniformLocation(program, "u_diffuse");
    var u_specularLoc  = gl.getUniformLocation(program, "u_specular");
    var u_shininessLoc = gl.getUniformLocation(program, "u_shininess");
    var u_specularFactorLoc= gl.getUniformLocation(program, "u_specularFactor");
    
    var a_positionLoc  = gl.getAttribLocation(program, "a_position");
    var a_normalLoc= gl.getAttribLocation(program, "a_normal");
    var a_texCoordLoc  = gl.getAttribLocation(program, "a_texcoord");
    
    
    // At init or draw time depending on use.
    var someWorldViewProjectionMat = computeWorldViewProjectionMatrix();
    var lightWorldPos  = [100, 200, 300];
    var worldMat   = computeWorldMatrix();
    var viewInverseMat = computeInverseViewMatrix();
    var worldInverseTransposeMat   = computeWorldInverseTransposeMatrix();
    var lightColor = [1, 1, 1, 1];
    var ambientColor   = [0.1, 0.1, 0.1, 1];
    var diffuseTextureUnit = 0;
    var specularColor  = [1, 1, 1, 1];
    var shininess  = 60;
    var specularFactor = 1;
    
    
    // At draw time
    gl.useProgram(program);
    
    // Setup all the buffers and attributes
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
    gl.enableVertexAttribArray(a_positionLoc);
    gl.vertexAttribPointer(a_positionLoc, positionNumComponents, gl.FLOAT, false, 0, 0);
    gl.bindBuffer(gl.ARRAY_BUFFER, normalBuffer);
    gl.enableVertexAttribArray(a_normalLoc);
    gl.vertexAttribPointer(a_normalLoc, normalNumComponents, gl.FLOAT, false, 0, 0);
    gl.bindBuffer(gl.ARRAY_BUFFER, texcoordBuffer);
    gl.enableVertexAttribArray(a_texcoordLoc);
    gl.vertexAttribPointer(a_texcoordLoc, texcoordNumComponents, gl.FLOAT, 0, 0);
    
    // Setup the textures used
    gl.activeTexture(gl.TEXTURE0 + diffuseTextureUnit);
    gl.bindTexture(gl.TEXTURE_2D, diffuseTexture);
    
    // Set all the uniforms.
    gl.uniformMatrix4fv(u_worldViewProjectionLoc, false, someWorldViewProjectionMat);
    gl.uniform3fv(u_lightWorldPosLoc, lightWorldPos);
    gl.uniformMatrix4fv(u_worldLoc, worldMat);
    gl.uniformMatrix4fv(u_viewInverseLoc, viewInverseMat);
    gl.uniformMatrix4fv(u_worldInverseTransposeLoc, worldInverseTransposeMat);
    gl.uniform4fv(u_lightColorLoc, lightColor);
    gl.uniform4fv(u_ambientLoc, ambientColor);
    gl.uniform1i(u_diffuseLoc, diffuseTextureUnit);
    gl.uniform4fv(u_specularLoc, specularColor);
    gl.uniform1f(u_shininessLoc, shininess);
    gl.uniform1f(u_specularFactorLoc, specularFactor);
    
    gl.drawArrays(...);
    
这是大量的输入。  

这里有许多方法可以用来简化它。其中一项建议是要求 WebGL 告诉我们所有的制服和位置，然后设置函数，来帮助我们建立它们。然后我们可以通过   JavaScript 对象来使设置我们的设置更加容易。如果还是不清楚，我们的代码将会跟以下代码类似  

    // At initialiation time
    var uniformSetters = createUniformSetters(gl, program);
    var attribSetters  = createAttributeSetters(gl, program);
    
    var attribs = {
      a_position: { buffer: positionBuffer, numComponents: 3, },
      a_normal:   { buffer: normalBuffer,   numComponents: 3, },
      a_texcoord: { buffer: texcoordBuffer, numComponents: 2, },
    };
    
    // At init time or draw time depending on use.
    var uniforms = {
      u_worldViewProjection:   computeWorldViewProjectionMatrix(...),
      u_lightWorldPos: [100, 200, 300],
      u_world: computeWorldMatrix(),
      u_viewInverse:   computeInverseViewMatrix(),
      u_worldInverseTranspose: computeWorldInverseTransposeMatrix(),
      u_lightColor:[1, 1, 1, 1],
      u_ambient:   [0.1, 0.1, 0.1, 1],
      u_diffuse:   diffuseTexture,
      u_specular:  [1, 1, 1, 1],
      u_shininess: 60,
      u_specularFactor:1,
    };
    
    // At draw time
    gl.useProgram(program);
    
    // Setup all the buffers and    attributes
    setAttributes(attribSetters, attribs);
    
    // Set all the uniforms and textures used.
    setUniforms(uniformSetters, uniforms);
    
    gl.drawArrays(...);

这对于我来说，看起来是很多的更小，更容易，更少的代码。  
 
你甚至可以使用多个 JavaScript 对象，如果那样适合你的话。如下所示  

    // At initialiation time
    var uniformSetters = createUniformSetters(gl, program);
    var attribSetters  = createAttributeSetters(gl, program);
    
    var attribs = {
      a_position: { buffer: positionBuffer, numComponents: 3, },
      a_normal:   { buffer: normalBuffer,   numComponents: 3, },
      a_texcoord: { buffer: texcoordBuffer, numComponents: 2, },
    };
    
    // At init time or draw time depending
    var uniformsThatAreTheSameForAllObjects = {
      u_lightWorldPos: [100, 200, 300],
      u_viewInverse:   computeInverseViewMatrix(),
      u_lightColor:[1, 1, 1, 1],
    };
    
    var uniformsThatAreComputedForEachObject = {
      u_worldViewProjection:   perspective(...),
      u_world: computeWorldMatrix(),
      u_worldInverseTranspose: computeWorldInverseTransposeMatrix(),
    };
    
    var objects = [
      { translation: [10, 50, 100],
    materialUniforms: {
      u_ambient:   [0.1, 0.1, 0.1, 1],
      u_diffuse:   diffuseTexture,
      u_specular:  [1, 1, 1, 1],
      u_shininess: 60,
      u_specularFactor:1,
    },
      },
      { translation: [-120, 20, 44],
    materialUniforms: {
      u_ambient:   [0.1, 0.2, 0.1, 1],
      u_diffuse:   someOtherDiffuseTexture,
      u_specular:  [1, 1, 0, 1],
      u_shininess: 30,
      u_specularFactor:0.5,
    },
      },
      { translation: [200, -23, -78],
    materialUniforms: {
      u_ambient:   [0.2, 0.2, 0.1, 1],
      u_diffuse:   yetAnotherDiffuseTexture,
      u_specular:  [1, 0, 0, 1],
      u_shininess: 45,
      u_specularFactor:0.7,
    },
      },
    ];
    
    // At draw time
    gl.useProgram(program);
    
    // Setup the parts that are common for all objects
    setAttributes(attribSetters, attribs);
    setUniforms(uniformSetters, uniformThatAreTheSameForAllObjects);
    
    objects.forEach(function(object) {
      computeMatricesForObject(object, uniformsThatAreComputedForEachObject);
      setUniforms(uniformSetters, uniformThatAreComputedForEachObject);
      setUniforms(unifromSetters, objects.materialUniforms);
      gl.drawArrays(...);
    });

这里有一个使用这些帮助函数的例子  


![](/images/webgl-less-code-more-fun1.png)
<a href="http://webglfundamentals.org/webgl/webgl-less-code-more-fun.html">点击这里可以显示一个单独的窗体</a>  

让我们向前更进一小步。在上面代码中，我们设置了一个拥有我们创建的缓冲区的变量 `attribs`。在代码中不显示设置这些缓冲区的代码。例如，如果你想要设置位置，法线和纹理坐标，你可能会需要这样的代码  

    // a single triangle
    var positions = [0, -10, 0, 10, 10, 0, -10, 10, 0];
    var texcoords = [0.5, 0, 1, 1, 0, 1];
    var normals   = [0, 0, 1, 0, 0, 1, 0, 0, 1];
     
    var positionBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
     
    var texcoordBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, texcoordsBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(texcoords), gl.STATIC_DRAW);
     
    var normalBuffer = gl.createBuffer();
    gl.bindBuffer(gl.ARRAY_BUFFER, normalBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(normals), gl.STATIC_DRAW);

看起来像一种我们也可以简化的模式。

     // a single triangle
    var arrays = {
       position: { numComponents: 3, data: [0, -10, 0, 10, 10, 0, -10, 10, 0], },
       texcoord: { numComponents: 2, data: [0.5, 0, 1, 1, 0, 1],   },
       normal:   { numComponents: 3, data: [0, 0, 1, 0, 0, 1, 0, 0, 1],},
    };
     
    var bufferInfo = createBufferInfoFromArrays(gl, arrays); 
    
更短！现在我们可以在渲染时间这样做  

    // Setup all the needed buffers and attributes.
    setBuffersAndAttributes(gl, attribSetters, bufferInfo);
     
    ...
     
    // Draw the geometry.
    gl.drawArrays(gl.TRIANGLES, 0, bufferInfo.numElements);

如下所示  

![](/images/webgl-less-code-more-fun2.png)

<a href="http://webglfundamentals.org/webgl/webgl-less-code-more-fun-triangle.html">点击这里可以显示一个单独的窗体</a>   
  
如果我们有 indices，这可能会奏效。setAttribsAndBuffers 将会设置所有的属性，同时用你的 `indices` 来设置 `ELEMENT-ARRAY-BUFFER`。 所以你可以调用 `gl.drawElements`.  

    // an indexed quad
    var arrays = {
       position: { numComponents: 3, data: [0, 0, 0, 10, 0, 0, 0, 10, 0, 10, 10, 0], },
       texcoord: { numComponents: 2, data: [0, 0, 0, 1, 1, 0, 1, 1], },
       normal:   { numComponents: 3, data: [0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1], },
       indices:  { numComponents: 3, data: [0, 1, 2, 1, 2, 3],   },
    };
     
    var bufferInfo = createBufferInfoFromTypedArray(gl, arrays);

同时在渲染时，我们可以调用 `gl.drawElements`，而不是  `gl.drawArrays`。  

    // Setup all the needed buffers and attributes.
    setBuffersAndAttributes(gl, attribSetters, bufferInfo);
     
    ...
     
    // Draw the geometry.
    gl.drawElements(gl.TRIANGLES, bufferInfo.numElements, gl.UNSIGNED_SHORT, 0);

如下所示  

![](/images/webgl-less-code-more-fun3.png)
<a href="http://webglfundamentals.org/webgl/webgl-less-code-more-fun-quad.html">点击这里可以显示一个单独的窗体</a>

`createBufferInfoFromArrays` 基本上使一个对象与如下代码相似  

     bufferInfo = {
       numElements: 4,// or whatever the number of elements is
       indices: WebGLBuffer,  // this property will not exist if there are no indices
       attribs: {
     a_position: { buffer: WebGLBuffer, numComponents: 3, },
     a_normal:   { buffer: WebGLBuffer, numComponents: 3, },
     a_texcoord: { buffer: WebGLBuffer, numComponents: 2, },
       },
     };
    
同时 `setBuffersAndAttributes` 使用这个对象来设置所有的缓冲区和属性。  

最后我们可以进展到我之前认为可能太远的地步。给出的 `position` 几乎总是拥有 3 个组件 (x, y, z)，同时 `texcoords` 几乎总是拥有 2 个组件，indices 几乎总是有 3 个组件，同时 normals 总是有 3 个组件，我们就可以让系统来猜想组件的数量。  

    // an indexed quad
    var arrays = {
       position: [0, 0, 0, 10, 0, 0, 0, 10, 0, 10, 10, 0],
       texcoord: [0, 0, 0, 1, 1, 0, 1, 1],
       normal:   [0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 1],
       indices:  [0, 1, 2, 1, 2, 3],
    };

以下是另一个版本。    

![](/images/webgl-less-code-more-fun4.png)
<a href="http://webglfundamentals.org/webgl/webgl-less-code-more-fun-quad-guess.html">点击这里可以显示一个单独的窗体</a>

我不确认我个人喜欢那种版本。我可能猜测出错，因为它可能猜错。例如，我可能选择在我的 texcoord 属性中添加额外的一组结构坐标，然后它会猜测 2，是错误的。当然，如果它猜错了，你可以像以上示例中那样指定个数。我想我担心的，如果猜测代码改变了人们的日常的情况可能会打破。全都取决于你。一些人喜欢让东西尽量像他们考虑的那样简单。  

我们为什么不在着色器程序中查看这些属性来得出组件的数量？那是因为，从一个缓冲区中提供 3 组件 (x, y, z)，但是在着色器中使用 `vec4` 是非常普遍的 。对于属性 WebGL 会自动设置 `w = 1`.但是那意味着，我们不可能很容易的就知道用户的意图，因为他们在他们的着色器中声明的可能与他们提供的组件的数量不匹配。  

如果想要寻找更多的模式，如下所示  

    var program = createProgramFromScripts(gl, ["vertexshader", "fragmentshader"]);
    var uniformSetters = createUniformSetters(gl, program);
    var attribSetters  = createAttributeSetters(gl, program);

让我们将上述代码简化成如下代码  


    var programInfo = createProgramInfo(gl, ["vertexshader", "fragmentshader"]);

它将返回与下面代码类似的东西

    programInfo = {
       program: WebGLProgram,  // program we just compiled
       uniformSetters: ...,// setters as returned from createUniformSetters,
       attribSetters: ..., // setters as returned from createAttribSetters,
    }
    
那是另一个更小的简化。在我们开始使用多个程序时，它将会派上用场，因为它自动保持与它们的相关联的程序的设定。  

![](/images/webgl-less-code-more-fun5.png)
<a href="http://webglfundamentals.org/webgl/webgl-less-code-more-fun-quad-programinfo.html">点击这里可以显示一个单独的窗体</a> 

无论如何，这是我想要编写我自己的 WebGL 程序的风格。在这些教程中的课程中，尽管我已经感觉到我需要使用标准的 **verbose** 方法，这样人们就不会对 WebGL 是什么和什么是我自己的风格感到困惑。在一些点上，尽管显示所有的可以获取这些点的方式的步骤，所以继续学习这些课程的可以在这种风格中被使用。  
 
在你自己的代码中随便使用这种风格。 `createUniformSetters`,  `createAttributeSetters`, `createBufferInfoFromArrays`, `setUniforms` 和 `setBuffersAndAttributes` 这些函数都包含在 <a href="https://github.com/greggman/webgl-fundamentals/blob/master/webgl/resources/webgl-utils.js">webgl-utils.js</a> 文件中，可以在所有的例子中使用。如果你想要一些更多有组织的东西，可以查看 <a href="http://twgljs.org/">TWGL.js</a>。  


接下来，<a href="http://webglfundamentals.org/webgl/lessons/webgl-drawing-multiple-things.html">绘制多个事物</a>。  
