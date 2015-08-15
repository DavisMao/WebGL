##WebGL 2D 图像旋转##

这篇是关于 WebGL 系列中的一篇。第一篇是[基础知识讲解](http://webglfundamentals.org/webgl/lessons/webgl-fundamentals.html)，本文的前一篇是[移动几何图形](http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html)。

首先我要承认下我不是很清楚如何讲解这个让它看起来更容易理解，但是，不管怎么样，我想尽力尝试下。首先，我想介绍下什么叫做“单位圆”。你如果还记得高中数学中(不要睡着了！)一个圆又一个半径。半径指的是从圆心到圆的圆边的距离。单位圆指的是它的半径是 1.0。

如下是一个单位圆：

![unit circle](/images/unit_circle.png)

[打开一个单独的窗口尝试下单位圆](http://webglfundamentals.org/webgl/unit-circle.html)

打开上面的链接之后，你可以拖动圆环上面的小圆，接着 X 和 Y 的值也会随之发生变化。这个左边值表示的是圆环上的点。在圆上的最高点处，Y 为 1 和 X 为 0。在最右的位置时 X 为 1 和 Y 为 0。

如果你还记得基础的三年级数学，把某个数乘以 1 以后结果仍然是该数。那么 123*1 = 123。相当基础对吧？那么半径为 1.0 的单位圆也是一种形式的  1。它是一种旋转的 1。因此你可以将这个单位圆与某物相乘，它执行的操作和乘以 1 类似，除了一些奇异的事情发生改变这种方式。

我们将从单位圆上得到任何点的 X 和 Y 值，接着将他们乘以[上一节示例](http://webglfundamentals.org/webgl/lessons/webgl-2d-translation.html)中的几何图形。

如下是更新渲染器：

	<script id="2d-vertex-shader" type="x-shader/x-vertex">
	attribute vec2 a_position;
 
	uniform vec2 u_resolution;
	uniform vec2 u_translation;
	uniform vec2 u_rotation;
 
	void main() {
 	 // Rotate the position
  	vec2 rotatedPosition = vec2(
     a_position.x * u_rotation.y + a_position.y * u_rotation.x,
     a_position.y * u_rotation.y - a_position.x * u_rotation.x);
 
  	// Add in the translation.
  	vec2 position = rotatedPosition + u_translation;

接着修改 JavaScript 代码，这样我们就可以传递上面的两个参数：

```
...
  var rotationLocation = gl.getUniformLocation(program, "u_rotation");
  ...
  var rotation = [0, 1];
  ..
  // Draw the scene.
  function drawScene() {
    // Clear the canvas.
    gl.clear(gl.COLOR_BUFFER_BIT);
 
    // Set the translation.
    gl.uniform2fv(translationLocation, translation);
 
    // Set the rotation.
    gl.uniform2fv(rotationLocation, rotation);
 
    // Draw the rectangle.
    gl.drawArrays(gl.TRIANGLES, 0, 18);
  }
```

如下是代码运行的结果。拖动单位圆上的小环使图形进行旋转或者拖动滑动条使图形进行移动。

![rotation geometry](/images/webgl-2d-rotation-geometry.png)

[点我打开一个单独的窗口尝试下](http://webglfundamentals.org/webgl/webgl-2d-geometry-rotation.html)

为什么上面的代码能够起作用？首先，让我们看下数学公式：

```
rotatedX = a_position.x * u_rotation.y + a_position.y * u_rotation.x;
rotatedY = a_position.y * u_rotation.y - a_position.x * u_rotation.x;
```

假设你有一个矩形，并且你想旋转它。在你把它旋转到右上角 (3.0，9.0) 这个位置之前。我们先在单位圆中选择一个从 12 点钟的位置顺时针偏移 30 度的点。

![30 degrees clockwise from 12 o'clock ](/images/clockwise_30_degree.png)

在圆上那个位置的点的坐标为 0.50 和 0.87：

	3.0 * 0.87 + 9.0 * 0.50 = 7.1
	9.0 * 0.87 - 3.0 * 0.50 = 6.3

那刚刚好是我们需要的位置：

![rotate rectangle](/images/rotate_rectangle.png)

旋转 60 度和上面的操作一样：

![rotate 60 degrees](/images/clockwise_60_degree.png)

圆上面的位置的坐标是 0.87 和 0.50:

  	3.0 * 0.50 + 9.0 * 0.87 = 9.3
 	9.0 * 0.50 - 3.0 * 0.87 = 1.9

你可以发现当我们顺时针向右旋转那个点时，X 的值变得更大而 Y 的值在变小。如果接着旋转超过 90 度，X 的值将再次变小而 Y 的值将变得更大。这个形式就能够达到旋转的目的。

圆环上的那些点还有另外一个名称。他们被称作为 sine 和 cosine。因此，对任意给定的角度，我们就只需查询它所对应的 sine 和 cosine 值：

```
function printSineAndCosineForAnyAngle(angleInDegrees) {
  var angleInRadians = angleInDegrees * Math.PI / 180;
  var s = Math.sin(angleInRadians);
  var c = Math.cos(angleInRadians);
  console.log("s = " + s + " c = " + c);
}
```

如果你把上面的代码复制粘贴到 JavaScript 控制台中，接着输入  `printSineAndCosineForAnyAngle(30)`，接着你会看到输出 `s = 0.49 c = 0.87`(注意：这个数字是近似值。)

如果你把上面的代码整合在一起的话，你就可以将你的几何体按照你想要的任何角度进行旋转。仅仅只需要将你需要旋转的角度值传给 sine 和 cosine 就可以了。

```
 ...
  var angleInRadians = angleInDegrees * Math.PI / 180;
  rotation[0] = Math.sin(angleInRadians);
  rotation[1] = Math.cos(angleInRadians);
```

如下是设置一个角度旋转的版本。拖动滑动条旋转或者移动。

![rotate angle](/images/rotate_angle.png)

[点我打开一个单独的窗口进行尝试](http://webglfundamentals.org/webgl/webgl-2d-geometry-rotation-angle.html)

希望上面讲的能够让你理解旋转的含义。下一节是简单点的[伸缩](http://webglfundamentals.org/webgl/lessons/webgl-2d-scale.html)。

> 什么是弧度？

> 弧度是一个测量单位，它通常与圆，旋转和角度一起使用。就像可以用英寸，码，米等来测量距离，弧度可以测量角度或者半径。
> 
> 你或许也意识到公制的测量单位比英制的测量的更简单。英寸除以 12 就得到英尺。英寸除以 36 就得到码。我不清楚你的情况，但是我不能在脑海中求出除以 36 的结果。对于公制它是更容易点的，毫米转化为厘米除以 10 就好，从毫米转换成米只需除以 1000 就好。我能默算出除以 1000 的结果。
> 
> 弧度相对于度来说更简单。度让数学计算比较难。弧度简化了数学计算。一个圆是 360 度，然而只有 2π 弧度。因此转一整圈是 2π 弧度。半圈是 1π 弧度。 1/4 圈是 90 度，1/2π弧度。因此你如果想要将某物旋转 90 度，仅仅只需使用 `Math.PI * 0.5`。如果你想旋转 45 度，你只需使用 `Math.PI * 0.25`等。
>
>几乎所有的数学都涉及到角度，圆或者旋转，如果使用弧度他们运用起来就非常简单。因此，推荐尝试下使用弧度而不是度，除了在 UI 中显示之外。
