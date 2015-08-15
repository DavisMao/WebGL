##WebGL 2D 图像 Scale##

这篇文章仍然属于讲解 WebGL 系列的文章。第一篇是[基础知识讲解](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)，本文的前一篇是[几何图形旋转](http://webglfundamentals.org/webgl/lessons/webgl-2d-rotation.html)。

图像伸缩和[转换](http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html)一样简单。我们只需对需要变换的点乘以我们想要的比例。如下是从[以前的代码](http://webglfundamentals.org/webgl/lessons/webgl-2d-rotation.html)改变而来的。

	<script id="2d-vertex-shader" type="x-shader/x-vertex">
	attribute vec2 a_position;
 
	uniform vec2 u_resolution;
	uniform vec2 u_translation;
	uniform vec2 u_rotation;
	uniform vec2 u_scale;
 
	void main() {
  	// Scale the positon
  	vec2 scaledPosition = a_position * u_scale;
 
  	// Rotate the position
  	vec2 rotatedPosition = vec2(
     scaledPosition.x * u_rotation.y + scaledPosition.y * u_rotation.x,
     scaledPosition.y * u_rotation.y - scaledPosition.x * u_rotation.x);
 
  	// Add in the translation.
  	vec2 position = rotatedPosition + u_translation;

接着当我们需要绘图时添加必要的 JavaScript 代码来设置伸缩比例。

```
 ...
  var scaleLocation = gl.getUniformLocation(program, "u_scale");
  ...
  var scale = [1, 1];
  ...
  // Draw the scene.
  function drawScene() {
    // Clear the canvas.
    gl.clear(gl.COLOR_BUFFER_BIT);
 
    // Set the translation.
    gl.uniform2fv(translationLocation, translation);
 
    // Set the rotation.
    gl.uniform2fv(rotationLocation, rotation);
 
    // Set the scale.
    gl.uniform2fv(scaleLocation, scale);
 
    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 18);
  }
```

现在我们就可以通过拖动滑动条对图像进行伸缩变换。

![geometry scale](/images/geometry_scale.png)

[点我打开一个单独的窗口尝试下](http://webglfundamentals.org/webgl/webgl-2d-geometry-scale.html)

需要注意的一件事是如果设置伸缩比例为负值，那么几何图形就会发生翻转。

希望这相邻的三篇文章能够帮助你理解图形转换，旋转和伸缩。接下来我们将讲解拥有魔力的矩阵，它能够很容易的将这三种操作集合在一起，而且通常是更有用的形式。

>为什么使用 ‘F’？
>我是在纹理上第一次看见别人使用 ‘F’。‘F’ 符号本身是不重要的。重要的是你可以从任何方向说明它的朝向。如果我们使用一个心形或者三角形的话，我们就不能区别该图形是否发生了水平翻转。一个圆圈的情况就更差了。可以证明四个角着不同颜色的矩形能够正常工作，但是你必须记住每个角上的颜色。F 的朝向是从它本身就可以看出来，是固有的。

![F orientation](/images/f-orientation.png)

任何你能够从它本身指定方向的图形都适用，我一直使用 ‘F’ 从我第一次意识到这一特点。
