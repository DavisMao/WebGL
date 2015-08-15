## WebGL 文本 - 使用字符纹理

这篇文章是前面许多 WebGL 文章的延续。上一篇文章是关于如何[在 WebGL 使用纹理渲染文本](http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html)。如果你还没有读过，您可能想要在继续之前查看一下。

在上一篇文章中我们复习了[在 WebGL 场景中如何使用纹理绘制文本](http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html)。技术是很常见的，对一些事物也是极重要的，例如在多人游戏中你想在一个头像上放置一个名字。同时这个名字也不能影响它的完美性。

比方说你想呈现大量的文本，这需要经常改变 UI 之类的事物。[前一篇文章](http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html)给出的最后一个例子中，一个明显的解决方案是给每个字母加纹理。我们来尝试一下改变上一个例子。

    var names = [
      "anna",   // 0
      "colin",  // 1
      "james",  // 2
      "danny",  // 3
      "kalin",  // 4
      "hiro",   // 5
      "eddie",  // 6
      "shu",// 7
      "brian",  // 8
      "tami",   // 9
      "rick",   // 10
      "gene",   // 11
      "natalie",// 12,
      "evan",   // 13,
      "sakura", // 14,
      "kai",// 15,
    ];
     
    // create text textures, one for each letter
    var textTextures = [
      "a",// 0
      "b",// 1
      "c",// 2
      "d",// 3
      "e",// 4
      "f",// 5
      "g",// 6
      "h",// 7
      "i",// 8
      "j",// 9
      "k",// 10
      "l",// 11
      "m",// 12,
      "n",// 13,
      "o",// 14,
      "p",// 14,
      "q",// 14,
      "r",// 14,
      "s",// 14,
      "t",// 14,
      "u",// 14,
      "v",// 14,
      "w",// 14,
      "x",// 14,
      "y",// 14,
      "z",// 14,
    ].map(function(name) {
      var textCanvas = makeTextCanvas(name, 10, 26);

相对于为每个名字呈现一个四元组，我们将为每个名字的每个字母呈现一个四元组。

    // setup to draw the text.
    // Because every letter uses the same attributes and the same progarm
    // we only need to do this once.
    gl.useProgram(textProgramInfo.program);
    setBuffersAndAttributes(gl, textProgramInfo.attribSetters, textBufferInfo);
     
    textPositions.forEach(function(pos, ndx) {
      var name = names[ndx];
      // for each leter
      for (var ii = 0; ii < name.length; ++ii) {
    var letter = name.charCodeAt(ii);
    var letterNdx = letter - "a".charCodeAt(0);
    // select a letter texture
    var tex = textTextures[letterNdx];
     
    // use just the position of the 'F' for the text
     
    // because pos is in view space that means it's a vector from the eye to
    // some position. So translate along that vector back toward the eye some distance
    var fromEye = normalize(pos);
    var amountToMoveTowardEye = 150;  // because the F is 150 units long
    var viewX = pos[0] - fromEye[0] * amountToMoveTowardEye;
    var viewY = pos[1] - fromEye[1] * amountToMoveTowardEye;
    var viewZ = pos[2] - fromEye[2] * amountToMoveTowardEye;
    var desiredTextScale = -1 / gl.canvas.height;  // 1x1 pixels
    var scale = viewZ * desiredTextScale;
     
    var textMatrix = makeIdentity();
    textMatrix = matrixMultiply(textMatrix, makeTranslation(ii, 0, 0));
    textMatrix = matrixMultiply(textMatrix, makeScale(tex.width * scale, tex.height * scale, 1));
    textMatrix = matrixMultiply(textMatrix, makeTranslation(viewX, viewY, viewZ));
    textMatrix = matrixMultiply(textMatrix, projectionMatrix);
     
    // set texture uniform
    textUniforms.u_texture = tex.texture;
    copyMatrix(textMatrix, textUniforms.u_matrix);
    setUniforms(textProgramInfo.uniformSetters, textUniforms);
     
    // Draw the text.
    gl.drawElements(gl.TRIANGLES, textBufferInfo.numElements, gl.UNSIGNED_SHORT, 0);
      }
    });

你可以看到它是如何工作的：

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-glyphs.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-glyphs.html" target="_blank">点击这里打开一个独立窗口</a>
</div>
</p>

不幸的是它很慢。下面的例子：单独绘制 73 个四元组，还看不出来差别。我们计算 73 个矩阵和 292 个矩阵倍数。一个典型的 UI 可能有 1000 个字母要显示。这是众多工作可以得到一个合理的帧速率的方式。

解决这个问题通常的方法是构造一个纹理图谱，其中包含所有的字母。我们讨论[给立方体的 6 面加纹理](http://webglfundamentals.org/webgl/lessons/webgl-3d-textures.html)时，复习了纹理图谱。

下面的代码构造了字符的纹理图谱。

    function makeGlyphCanvas(ctx, maxWidthOfTexture, heightOfLetters, baseLine, padding, letters) {
      var rows = 1;  // number of rows of glyphs
      var x = 0; // x position in texture to draw next glyph
      var y = 0; // y position in texture to draw next glyph
      var glyphInfos = { // info for each glyph
      };
     
      // Go through each letter, measure it, remember its width and position
      for (var ii = 0; ii < letters.length; ++ii) {
    var letter = letters[ii];
    var t = ctx.measureText(letter);
    // Will this letter fit on this row?
    if (x + t.width + padding > maxWidthOfTexture) {
       // so move to the start of the next row
       x = 0;
       y += heightOfLetters;
       ++rows;
    }
    // Remember the data for this letter
    glyphInfos[letter] = {
      x: x,
      y: y,
      width: t.width,
    };
    // advance to space for next letter.
    x += t.width + padding;
      }
     
      // Now that we know the size we need set the size of the canvas
      // We have to save the canvas settings because changing the size
      // of a canvas resets all the settings
      var settings = saveProperties(ctx);
      ctx.canvas.width = (rows == 1) ? x : maxWidthOfTexture;
      ctx.canvas.height = rows * heightOfLetters;
      restoreProperties(settings, ctx);
     
      // Draw the letters into the canvas
      for (var ii = 0; ii < letters.length; ++ii) {
    var letter = letters[ii];
    var glyphInfo = glyphInfos[letter];
    var t = ctx.fillText(letter, glyphInfo.x, glyphInfo.y + baseLine);
      }
     
      return glyphInfos;
    }

现在我们试试看：

    var ctx = document.createElement("canvas").getContext("2d");
    ctx.font = "20px sans-serif";
    ctx.fillStyle = "white";
    var maxTextureWidth = 256;
    var letterHeight = 22;
    var baseline = 16;
    var padding = 1;
    var letters = "0123456789.abcdefghijklmnopqrstuvwxyz";
    var glyphInfos = makeGlyphCanvas(
    ctx,
    maxTextureWidth,
    letterHeight,
    baseline,
    padding,
    letters);

结果如下

<p><div>
  <iframe class="webgl_example" style="width: 258px; height: 46px;" src="http://webglfundamentals.org/webgl/glyph-texture-atlas-maker.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/glyph-texture-atlas-maker.html" target="_blank">点击这里打开一个独立窗口</a>
</div>
</p>

现在，我们已经创建了一个我们需要使用的字符纹理。看看效果怎样，我们为每个字符建四个顶点。这些顶点将使用纹理坐标来选择特殊的字符。

给定一个字符串，来建立顶点：

    function makeVerticesForString(fontInfo, s) {
      var len = s.length;
      var numVertices = len * 6;
      var positions = new Float32Array(numVertices * 2);
      var texcoords = new Float32Array(numVertices * 2);
      var offset = 0;
      var x = 0;
      for (var ii = 0; ii < len; ++ii) {
    var letter = s[ii];
    var glyphInfo = fontInfo.glyphInfos[letter];
    if (glyphInfo) {
      var x2 = x + glyphInfo.width;
      var u1 = glyphInfo.x / fontInfo.textureWidth;
      var v1 = (glyphInfo.y + fontInfo.letterHeight) / fontInfo.textureHeight;
      var u2 = (glyphInfo.x + glyphInfo.width) / fontInfo.textureWidth;
      var v2 = glyphInfo.y / fontInfo.textureHeight;
     
      // 6 vertices per letter
      positions[offset + 0] = x;
      positions[offset + 1] = 0;
      texcoords[offset + 0] = u1;
      texcoords[offset + 1] = v1;
     
      positions[offset + 2] = x2;
      positions[offset + 3] = 0;
      texcoords[offset + 2] = u2;
      texcoords[offset + 3] = v1;
     
      positions[offset + 4] = x;
      positions[offset + 5] = fontInfo.letterHeight;
      texcoords[offset + 4] = u1;
      texcoords[offset + 5] = v2;
     
      positions[offset + 6] = x;
      positions[offset + 7] = fontInfo.letterHeight;
      texcoords[offset + 6] = u1;
      texcoords[offset + 7] = v2;
     
      positions[offset + 8] = x2;
      positions[offset + 9] = 0;
      texcoords[offset + 8] = u2;
      texcoords[offset + 9] = v1;
     
      positions[offset + 10] = x2;
      positions[offset + 11] = fontInfo.letterHeight;
      texcoords[offset + 10] = u2;
      texcoords[offset + 11] = v2;
     
      x += glyphInfo.width;
      offset += 12;
    } else {
      // we don't have this character so just advance
      x += fontInfo.spaceWidth;
    }
      }
     
      // return ArrayBufferViews for the portion of the TypedArrays
      // that were actually used.
      return {
    arrays: {
      position: new Float32Array(positions.buffer, 0, offset),
      texcoord: new Float32Array(texcoords.buffer, 0, offset),
    },
    numVertices: offset / 2,
      };
    }

为了使用它，我们手动创建一个 bufferInfo。([如果你已经不记得了，可以查看前面的文章：bufferInfo 是什么](http://webglfundamentals.org/webgl/lessons/webgl-drawing-multiple-things))。

    // Maunally create a bufferInfo
    var textBufferInfo = {
      attribs: {
    a_position: { buffer: gl.createBuffer(), numComponents: 2, },
    a_texcoord: { buffer: gl.createBuffer(), numComponents: 2, },
      },
      numElements: 0,
    };

使用 bufferInfo 中的字符创建画布的 fontInfo 和纹理：

    var ctx = document.createElement("canvas").getContext("2d");
    ctx.font = "20px sans-serif";
    ctx.fillStyle = "white";
    var maxTextureWidth = 256;
    var letterHeight = 22;
    var baseline = 16;
    var padding = 1;
    var letters = "0123456789.,abcdefghijklmnopqrstuvwxyz";
    var glyphInfos = makeGlyphCanvas(
    ctx,
    maxTextureWidth,
    letterHeight,
    baseline,
    padding,
    letters);
    var fontInfo = {
      glyphInfos: glyphInfos,
      letterHeight: letterHeight,
      baseline: baseline,
      spaceWidth: 5,
      textureWidth: ctx.canvas.width,
      textureHeight: ctx.canvas.height,
    };

然后渲染我们将更新缓冲的文本。我们也可以构成动态的文本：

    textPositions.forEach(function(pos, ndx) {
     
      var name = names[ndx];
      var s = name + ":" + pos[0].toFixed(0) + "," + pos[1].toFixed(0) + "," + pos[2].toFixed(0);
      var vertices = makeVerticesForString(fontInfo, s);
     
      // update the buffers
      textBufferInfo.attribs.a_position.numComponents = 2;
      gl.bindBuffer(gl.ARRAY_BUFFER, textBufferInfo.attribs.a_position.buffer);
      gl.bufferData(gl.ARRAY_BUFFER, vertices.arrays.position, gl.DYNAMIC_DRAW);
      gl.bindBuffer(gl.ARRAY_BUFFER, textBufferInfo.attribs.a_texcoord.buffer);
      gl.bufferData(gl.ARRAY_BUFFER, vertices.arrays.texcoord, gl.DYNAMIC_DRAW);
     
      setBuffersAndAttributes(gl, textProgramInfo.attribSetters, textBufferInfo);
     
      // use just the position of the 'F' for the text
      var textMatrix = makeIdentity();
      // because pos is in view space that means it's a vector from the eye to
      // some position. So translate along that vector back toward the eye some distance
      var fromEye = normalize(pos);
      var amountToMoveTowardEye = 150;  // because the F is 150 units long
      textMatrix = matrixMultiply(textMatrix, makeTranslation(
      pos[0] - fromEye[0] * amountToMoveTowardEye,
      pos[1] - fromEye[1] * amountToMoveTowardEye,
      pos[2] - fromEye[2] * amountToMoveTowardEye));
      textMatrix = matrixMultiply(textMatrix, projectionMatrix);
     
      // set texture uniform
      copyMatrix(textMatrix, textUniforms.u_matrix);
      setUniforms(textProgramInfo.uniformSetters, textUniforms);
     
      // Draw the text.
      gl.drawArrays(gl.TRIANGLES, 0, vertices.numVertices);
    });

即：

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-glyphs-texture-atlas.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-glyphs-texture-atlas.html" target="_blank">点击这里打开一个独立窗口</a>
</div>
</p>

这是使用字符纹理集的基本技术。可以添加一些明显的东西或方式来改进它。

- 重用相同的数组。   
  目前，每次被调用时，**makeVerticesForString** 就会分配新的 32 位浮点型数组。这最终可能会导致垃圾收集出现问题。重用相同的数组可能会更好。如果不是足够大，你也放大数组，但是保留原来的大小。
- 添加支持回车   
  当生成顶点时，检查 **\n** 是否存在从而实现换行。这将使文本分隔段落更容易。
- 添加对各种格式的支持。   
  如果你想文本居中，或调整你添加的一切文本的格式。
- 添加对顶点颜色的支持。   
  你可以为文本的每个字母添加不同的颜色。当然你必须决定如何指定何时改变颜色。

这里不打算涉及的另一个大问题是：纹理大小有限，但字体实际上是无限的。如果你想支持所有的 unicode，你就必须处理汉语、日语和阿拉伯语等其他所有语言，2015 年在 unicode 有超过 110000 个符号！你不可能在纹理中适配所有这些，也没有足够的空间供你这样做。

操作系统和浏览器 GPU 加速处理这个问题的方式是：通过使用一个字符纹理缓存实现。上面的实现他们是把纹理处理成纹理集，但他们为每个 glpyh 布置一个固定大小的区域，保留纹理集中最近使用的符号。如果需要绘制一个字符，而这个字符不在纹理集中，他们就用他们需要的这个新的字符取代最近最少使用的一个。当然如果他们即将取代的字符仍被有待绘制的四元组引用，他们需要绘制他们之前所取代的字符。

虽然我不推荐它，但是还有另一件事你可以做，将这项技术和[以前的技术](http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html)结合在一起。你可以直接渲染另一种纹理的符号。当然 GPU 加速画布已经这样做了，你可能没有自己动手的理由。

另一种在 WebGL 中绘制文本的方法实际上是使用了 3D 文本。在上面所有的例子中 “F” 是一个 3D 的字母。你已经为每个字母都构成了一个相应的 3D 字符。3D 字母常见于标题和电影标志，此外的用处就少了。

我希望在 WebGL 这可以覆盖文本。

### 使用 Canvas2D api 构造符号问题

当构造符号时，我该怎么决定 16 的 **baseline** 和 22 的 **letterHeight**？这实际上是一个我可以找出点问题的地方。问题是 HTML5 和画布 API 使我们无法了解这些事情。如果不查找每个字母，也就没有办法断定一个字体最高的字符有多高。而他们有 110,000 + 之多。

从 HTML5 中没有办法找出字体的基线，因此在这种字体绘制每一个字母将适于一个特定的矩形。

理想情况下你要知道，如果字体的基线是 16，那么高于基线，即超过 16 像素就什么也不能绘制，你还要知道最长下降者远低于基线。因为毫无办法从 HTML5 中得到信息，我们必须尝试不同的值并观察， 或者，我们在一个画布中必须一次绘制一个字母，然后扫描像素获得这个信息。这两个不是最好的解决方案。

希望当权者能添加一些新的 api 使这一切成为可能。

问题？[可以在 stackoverflow 提问](http://stackoverflow.com/questions/tagged/webgl)。
问题/缺陷？[可以在 github 创造一个话题](https://github.com/greggman/webgl-fundamentals/issues)。
