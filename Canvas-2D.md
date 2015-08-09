## WebGL 文本 - Canvas2D

本文是之前[绘制文本的 WebGL 文章](http://webglfundamentals.org/webgl/lessons/webgl-text-html.html)的延续。如果你还没有阅读它们，我建议你从头开始，然后再回到现在的内容。

相对于使用 HTML 元素制作文本我们还可以使用另一种使用 2D 上下文的画布。不需要分析，我们就可以做一个猜测，这将比使用 DOM 快。当然也会变得相对不灵活。你可能不能得到所有的 CSS 样式。但是，这儿没有 HTML 元素可以创建和跟踪。

和前边其他的例子类似，让我们来构造一个容器，但这一次我们将在其中放置两个画布。

    <div class="container">
      <canvas id="canvas" width="400" height="300"></canvas>
      <canvas id="text" width="400" height="300"></canvas>
    </div>

接下来设置 CSS，以使画布和 HTML 重叠

    .container {
    position: relative;
    }
     
    #text {
    position: absolute;
    left: 0px;
    top: 0px;
    z-index: 10;
    }

现在按照初始化时间查找文本画布，并为之创建一个 2D 上下文。

    // look up the text canvas.
    var textCanvas = document.getElementById("text");
     
    // make a 2D context for it
    var ctx = textCanvas.getContext("2d");

当绘图时，就像 WebGL，我们需要清除 2d 画布的每一帧。

    function drawScene() {
    ...
     
    // Clear the 2D canvas
    ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);

然后我们就调用 **fillText** 绘制文本

    ctx.fillText(someMsg, pixelX, pixelY);

下面有一个例子：

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-html-canvas2d.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-html-canvas2d.html" target="_blank">click here to open in a separate window</a>
</div>
</p>

为什么这个文本更小呢？因为这是 canvas2d 默认的尺寸。如果你想要其它的尺寸，可以[查看 canvas2d api](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Drawing_text)。

使用 canvas2d 的另一个原因是用它很容易绘制其他的事物。例如让我们来添加一个箭头：

    // draw an arrow and text.
     
    // save all the canvas settings
    ctx.save();
     
    // translate the canvas origin so 0, 0 is at
    // the top front right corner of our F
    ctx.translate(pixelX, pixelY);
     
    // draw an arrow
    ctx.beginPath();
    ctx.moveTo(10, 5);
    ctx.lineTo(0, 0);
    ctx.lineTo(5, 10);
    ctx.moveTo(0, 0);
    ctx.lineTo(15, 15);
    ctx.stroke();
     
    // draw the text.
    ctx.fillText(someMessage, 20, 20);
     
    // restore the canvas to its old settings.
    ctx.restore();

这里我们利用 canvas2d 的翻译功能，所以画箭头时，我们不需要做任何额外的工作。我们只是假装在原点开始绘制，翻译负责移动原点到 F 的角落。

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-html-canvas2d-arrows.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-html-canvas2d-arrows.html" target="_blank">点击这里打开一个独立的窗口</a>
</div>
</p>

封面使用了 2D 画布。[查看 canvas2d API](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D) 的更多想法。[接下来，我们将实际在 WebGL 呈现文本](http://webglfundamentals.org/webgl/lessons/webgl-text-texture.html)。

问题？[可以再 stackoverflow 提问](http://stackoverflow.com/questions/tagged/webgl)。

问题/缺陷？[可以在 github 上创建一个话题](https://github.com/greggman/webgl-fundamentals/issues)。