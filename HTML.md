## WebGL 文本 - HTML

这篇文章是先前 WebGL 系列文章的一个延续。如果你还没有阅读前面的内容，我建议你从[那里开始](http://webglfundamentals.org/webgl/lessons/webgl-3d-perspective.html)，然后再回到现在的内容。

“在 WebGL 中如何绘制文本”是一个我们常见的问题。那么第一件事就是我们要问自己绘制文本的目的何在。现在有一个浏览器，浏览器用来显示文本。所以你的第一个答案应该是如何使用 HTML 来显示文本。

让我们从最简单的例子开始：你只是想在你的 WebGL 上绘制一些文本。我们可以称之为一个文本覆盖。基本上这是停留在同一个位置的文本。

简单的方法是构造一些 HTML 元素，使用 CSS 使它们重叠。

例如：先构造一个容器，把画布和一些 HTML 元素重叠放置在容器内部。

    <div class="container">
      <canvas id="canvas" width="400" height="300"></canvas>
      <div id="overlay">
    <div>Time: <span id="time"></span></div>
    <div>Angle: <span id="angle"></span></div>
      </div>
    </div>

接下来设置 CSS，以达到画布和 HTML 重叠的目的。

    .container {
    position: relative;
    }
    #overlay {
    position: absolute;
    left: 10px;
    top: 10px;
    }

现在按照初始化和创建时间查找这些元素，或者查找你想要改变的区域。

    // look up the elements we want to affect
    var timeElement = document.getElementById("time");
    var angleElement = document.getElementById("angle");
     
    // Create text nodes to save some time for the browser.
    var timeNode = document.createTextNode("");
    var angleNode = document.createTextNode("");
     
    // Add those text nodes where they need to go
    timeElement.appendChild(timeNode);
    angleElement.appendChild(angleNode);

最后在渲染时更新节点。

    function drawScene() {
    ...
     
    // convert rotation from radians to degrees
    var angle = radToDeg(rotation[1]);
     
    // only report 0 - 360
    angle = angle % 360;
     
    // set the nodes
    angleNode.nodeValue = angle.toFixed(0);  // no decimal place
    timeNode.nodeValue = clock.toFixed(2);   // 2 decimal places

这里有一个例子:

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-html-overlay.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-html-overlay.html" target="_blank">点击这里打开一个独立的窗口</a>
</div>
</p>

注意为了我想改变的部分，我是如何把 spans 置入特殊的 div 内的。在这里我做一个假设，这比只使用 div 而没有 spans 速度要快，类似的有：

    timeNode.value = "Time " + clock.toFixed(2);

另外，我们可以使用文本节点，通过调用 **node = document.createTextNode()** 和 **laternode.node = someMsg**。我们也可以使用 **someElement.innerHTML = someHTML**。这将会更加灵活，虽然这样可能会稍微慢一些，但是您却可以插入任意的 HTML 字符串，因为你每次设置它，浏览器都不得不创建和销毁节点。这对你来说更加方便。

不采用叠加技术很重要的一点是，WebGL 在浏览器中运行。要记得在适当的时候使用浏览器的特征。大量的 OpenGL 程序员习惯于从一开始就 100% 靠他们自己实现应用的每一部分自己，因为 WebGL 要在一个已经有很多特征的浏览器上运行。使用它们有很多好处。例如使用 CSS 样式，你可以很容易就覆盖一个有趣的风格。

这里有一个相同的例子，但是添加了一些风格。背景是圆形的，字母的周围有光晕。这儿有一个红色的边界。你可以使用 HTML 免费得到所有。

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-html-overlay-styled.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-html-overlay-styled.html" target="_blank">点击这里打开一个独立的窗口</a>
</div>

</p>

我们要讨论的下一件最常见的事情是文本相对于你渲染的东西的位置。我们也可以在 HTML 中做到这一点。

在本例中，我们将再次构造一个画布容器和另一个活动的 HTML 容器。
    
    <div class="container">
      <canvas id="canvas" width="400" height="300"></canvas>
      <div id="divcontainer"></div>
    </div>

我们将设置 CSS

    .container {
    position: relative;
    overflow: none;
    }
     
    #divcontainer {
    position: absolute;
    left: 0px;
    top: 0px;
    width: 400px;
    height: 300px;
    z-index: 10;
    overflow: hidden;
     
    }
     
    .floating-div {
    position: absolute;
    }

相对于第一个 父类位置 **position: relative** 或者 **position: absolute** 风格，**position: absolute;** 部分使 **#divcontainer** 放置于绝对位置。在本例中，画布和 **#divcontainer** 都在容器内。

**left: 0px; top: 0px** 使 **#divcontainer** 结合一切。**z-index: 10** 使它浮在画布上。**overflow: hidden** 让它的子类被剪除。

最后，**floating-div** 将使用我们创建的可移式 div。

现在我们需要查找 div 容器，创建一个 div，将 div 附加到容器。

    // look up the divcontainer
    var divContainerElement = document.getElementById("divcontainer");
     
    // make the div
    var div = document.createElement("div");
     
    // assign it a CSS class
    div.className = "floating-div";
     
    // make a text node for its content
    var textNode = document.createTextNode("");
    div.appendChild(textNode);
     
    // add it to the divcontainer
    divContainerElement.appendChild(div);

现在，我们可以通过设置它的风格定位 div。
    
    div.style.left = Math.floor(x) + "px";
    div.style.top  = Math.floor(y) + "px";
    textNode.nodeValue = clock.toFixed(2);

下面是一个例子,我们只需要限制 div 的边界。

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-html-bouncing-div.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-html-bouncing-div.html" target="_blank">点击这里打开一个独立的窗口</a>
</div>

</p>

下一步，我们想在 3D 场景中设计它相对于某些事物的位置。我们该如何做？当我们[透视投影覆盖](http://webglfundamentals.org/webgl/lessons/webgl-3d-perspective.html)，我们如何做实际上就是我们请求 GPU 如何做。

通过这个例子我们学习了如何使用模型，如何复制它们，以及如何应用一个投影模型将他们转换成 clipspace。然后我们讨论着色器的内容，它在本地空间复制模型，并将其转换成 clipspace。我们也可以在 JavaScript 中做所有的这些。然后我们可以增加 clipspace(-1 到 +1) 到像素和使用 div 位置。

    gl.drawArrays(...);
     
    // We just got through computing a matrix to draw our
    // F in 3D.
     
    // choose a point in the local space of the 'F'.
    // X  Y  Z  W
    var point = [100, 0, 0, 1];  // this is the front top right corner
     
    // compute a clipspace position
    // using the matrix we computed for the F
    var clipspace = matrixVectorMultiply(point, matrix);
     
    // divide X and Y by W just like the GPU does.
    clipspace[0] /= clipspace[3];
    clipspace[1] /= clipspace[3];
     
    // convert from clipspace to pixels
    var pixelX = (clipspace[0] *  0.5 + 0.5) * gl.canvas.width;
    var pixelY = (clipspace[1] * -0.5 + 0.5) * gl.canvas.height;
     
    // position the div
    div.style.left = Math.floor(pixelX) + "px";
    div.style.top  = Math.floor(pixelY) + "px";
    textNode.nodeValue = clock.toFixed(2);

wahlah，我们 div 的左上角和 F 的右上角完全符合。

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-html-div.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-html-div.html" target="_blank">点击这里打开一个独立的窗口</a>
</div>

</p>

当然，如果你想要更多的文本构造更多的 div，如下：

<p><div>
  <iframe class="webgl_example" style="width: 400px; height: 300px;" src="http://webglfundamentals.org/webgl/webgl-text-html-divs.html"></iframe>
  <a class="webgl_center" href="http://webglfundamentals.org/webgl/webgl-text-html-divs.html" target="_blank">点击这里打开一个独立的窗口</a>
</div>

</p>

你可以查看最后一个例子的源代码来观察细节。一个重要的点是我猜测从 DOM 创建、添加、删除 HTML 元素是缓慢的，所以上面的示例用来创建它们，将它们保留在周围。它将任何未使用的都隐藏起来，而不是把他们从 DOM 删除。你必须明确知道是否可以更快。这只是我选择的方法。

希望这已经清楚地说明了如何使用 HTML 制造文本。[接下来，我们将介绍如何使用 Canvas2D 制造文本](http://webglfundamentals.org/webgl/lessons/webgl-text-canvas2d.html)。

问题？[可以在 stackoverflow 提问](http://stackoverflow.com/questions/tagged/webgl)。
问题/缺陷？[可以在 github 创造一个话题](https://github.com/greggman/webgl-fundamentals/issues)。