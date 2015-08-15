##WebGL 3D - 摄像机

这篇文章是一系列关于 WebGL 的文章的一个延续。首先是<a href="webgl-fundamentals.html">基本原理</a>以及前面的关于 <a href="webgl-3d-perspective.html">3D 透视投影</a>。如果你还没有读过，请先学习前面章节。

在过去的章节里我们将 F 移动到截锥的前面，因为 **makePerspective** 函数从原点（0，0，0）度量它，并且截锥的对象从 -zNear 到 -zFar 都在它前面。

视点前面移动的物体似乎没有正确的方式去做吗？在现实世界中，你通常会移动你的相机来给建筑物拍照。

<p align="center">
将摄像机移动到对象前
</p>

你通常不会将建筑移动到摄像机前。

<p align="center">
将对象移动到摄像机前
</p>

但在我们最后一篇文章中，我们提出了一个投影，这就需要物体在 Z 轴的原点前面。为了实现它，我们想做的是把摄像机移动到原点，然后把所有的其它物体都移动恰当的距离，所以它相对于摄像机仍然是在同一个地方。

<p align="center">
将对象移动到视图
</p>

我们需要有效地将现实中的物体移动到摄像机的前面。能达到这个目的的最简单的方法是使用“逆”矩阵。一般情况下的逆矩阵的计算是复杂的，但从概念上讲，它是容易的。逆是你用来作为其他数值的对立的值。例如，123 的是相反数是 -123。缩放比例为5的规模矩阵的逆是 1/5 或 0.2。在 X 域旋转30°的矩阵的逆是一个在 X 域旋转 -30° 的矩阵。

直到现在我们已经使用了平移，旋转和缩放来影响我们的 'F' 的位置和方向。把所有的矩阵相乘后，我们有一个单一的矩阵，表示如何将 “F” 以我们希望的大小和方向从原点移动到相应位置。使用摄像机我们可以做相同的事情。一旦我们的矩阵告诉我们如何从原点到我们想要的位置移动和旋转摄像机，我们就可以计算它的逆，它将给我们一个矩阵来告诉我们如何移动和旋转其它一切物体的相对数量，这将有效地使摄像机在点（0，0，0），并且我们已经将一切物体移动到它的前面。

让我们做一个有一圈 'F' 的三维场景，就像上面的图表那样。

下面是实现代码。

      var numFs = 5;
      var radius = 200;
     
      // Compute the projection matrix
      var aspect = canvas.clientWidth / canvas.clientHeight;
      var projectionMatrix =
          makePerspective(fieldOfViewRadians, aspect, 1, 2000);
     
      // Draw 'F's in a circle
      for (var ii = 0; ii < numFs; ++ii) {
        var angle = ii * Math.PI * 2 / numFs;
     
        var x = Math.cos(angle) * radius;
        var z = Math.sin(angle) * radius;
        var translationMatrix = makeTranslation(x, 0, z);
     
        // Multiply the matrices.
        var matrix = translationMatrix;
        matrix = matrixMultiply(matrix, projectionMatrix);
     
        // Set the matrix.
        gl.uniformMatrix4fv(matrixLocation, false, matrix);
     
        // Draw the geometry.
        gl.drawArrays(gl.TRIANGLES, 0, 16 * 6);
      }

就在我们计算出我们的投影矩阵之后，我们就可以计算出一个就像上面的图表中显示的那样围绕 ‘F’ 旋转的摄像机。

      // Compute the camera's matrix
      var cameraMatrix = makeTranslation(0, 0, radius * 1.5);
      cameraMatrix = matrixMultiply(
          cameraMatrix, makeYRotation(cameraAngleRadians));

然后，我们根据相机矩阵计算“视图矩阵”。“视图矩阵”是将一切物体移动到摄像机相反的位置，这有效地使摄像机相对于一切物体就像在原点（0,0,0）。

      // Make a view matrix from the camera matrix.
      var viewMatrix = makeInverse(cameraMatrix);

最后我们需要应用视图矩阵来计算每个 ‘F’ 的矩阵

        // Multiply the matrices.
        var matrix = translationMatrix;
        matrix = matrixMultiply(matrix, viewMatrix);  // <=-- added
        matrix = matrixMultiply(matrix, projectionMatrix);

一个摄像机可以绕着一圈 “F”。拖动 **cameraAngle** 滑块来移动摄像机。
<p align="center">
<a class="webgl_center" target="_blank" href="../webgl-3d-camera.html">点击这里在新窗口中打开</a>
</p>

这一切都很好，但使用旋转和平移来移动一个摄像头到你想要的地方，并且指向你想看到的地方并不总是很容易。例如如果我们想要摄像机总是指向特定的　‘F’　就要进行一些非常复杂的数学计算来决定当摄像机绕　‘F’　圈旋转的时候如何旋转摄像机来指向那个　‘F’。

幸运的是，有一个更容易的方式。我们可以决定摄像机在我们想要的地方并且可以决定它指向什么，然后计算矩阵，这个矩阵可以将把摄像机放到那里。基于矩阵的工作原理这非常容易实现。

首先，我们需要知道我们想要摄像机在什么位置。我们将称之为 **CameraPosition**。然后我们需要了解我们看过去或瞄准的物体的位置。我们将把它称为 **target**。如果我们将 **CameraPosition** 减去 **target** 我们将得到一个向量，它指向从摄像头获取目标的方向。让我们称它为 **zAxis**。因为我们知道摄像机指向 -Z 方向，我们可以从另一方向做减法 **cameraPosition - target**。我们将结果规范化，并直接复制到 **z** 区域矩阵。

<div class="webgl_math_center">
<pre class="webgl_math">
+----+----+----+----+
|    |    |    |    | 
+----+----+----+----+ 
|    |    |    |    | 
+----+----+----+----+ 
| Zx | Zy | Zz |    | 
+----+----+----+----+
|    |    |    |    | 
+----+----+----+----+ 
</pre>
</div>

这部分矩阵表示的是 Z 轴。在这种情况下，是摄像机的 Z 轴。一个向量的标准化意味着它代表了1.0。如果你回到<a href="webgl-2d-rotation.html">二维旋转</a>的文章，在哪里我们谈到了如何与单位圆以及二维旋转，在三维中我们需要单位球面和一个归一化的向量来代表在单位球面上一点。

<p align="center">
<span style="color:blue;">Z轴</span>
</p>

虽然没有足够的信息。只是一个单一的向量给我们一个点的单位范围内，但从这一点到东方的东西？我们需要把矩阵的其他部分填好。特别的X轴和Y轴类零件。我们知道这3个部分是相互垂直的。我们也知道，“一般”我们不把相机指向。因为，如果我们知道哪个方向是向上的，在这种情况下（0,1,0），我们可以使用一种叫做“跨产品和“计算X轴和Y轴的矩阵。

我不知道一个跨产品意味着在数学方面。我所知道的是，如果你有2个单位向量和你计算的交叉产品，你会得到一个向量，是垂直于这2个向量。换句话说，如果你有一个向量指向东南方，和一个向量指向上，和你计算交叉产品，你会得到一个向量指向北西或北东自这2个向量，purpendicular到东南亚和。根据你计算交叉产品的顺序，你会得到相反的答案。

在任何情况下，如果我们计算我们 **zAxis** 和 **up** 的相乘结果，我们会得到相机的 <span style="color:red;">xAxis</span>。



<div class="webgl_center">
<p align="center">
<span style="color:blue;">zAxis</span>
乘
<span style="color:gray;">up</span>
=
<span style="color:red;">xAxis</span>
</p>
</div>



现在，我们有  **xAxis**，我们可以通过 **zAxis** 和 **xAxis** 得到摄像机的 **yAxis**

现在我们所要做的就是将 3 个轴插入一个矩阵。这使得矩阵可以指向物体，从 **cameraPosition** 指向 **target**。我们只需要添加 **position**

<div class="webgl_math_center">
<pre class="webgl_math">
+----+----+----+----+ 
|<span style="color:red"> Xx </span>|<span style="color:red"> Xy </span>|<span style="color:red"> Xz </span>|  0 |  <- <span style="color:red">x axis</span>
+----+----+----+----+ 
|<span style="color:green"> Yx </span>|<span style="color:green"> Yy </span>|<span style="color:green"> Yz </span>|  0 |  <- <span style="color:green">y axis</span>
+----+----+----+----+ 
|<span style="color:blue"> Zx </span>|<span style="color:blue"> Zy </span>|<span style="color:blue"> Zz </span>|  0 |  <- <span style="color:blue">z axis</span>
+----+----+----+----+
| Tx | Ty | Tz |  1 |  <- camera position
+----+----+----+----+
</pre>
</div>

下面是用来计算2个向量的交叉乘积的代码。

    function cross(a, b) {
      return [a[1] * b[2] - a[2] * b[1],
              a[2] * b[0] - a[0] * b[2],
              a[0] * b[1] - a[1] * b[0]];
    }

这是减去两个向量的代码。

    function subtractVectors(a, b) {
      return [a[0] - b[0], a[1] - b[1], a[2] - b[2]];
    }

这里是规范化一个向量（使其成为一个单位向量）的代码。

    function normalize(v) {
      var length = Math.sqrt(v[0] * v[0] + v[1] * v[1] + v[2] * v[2]);
      // make sure we don't divide by 0.
      if (length > 0.00001) {
        return [v[0] / length, v[1] / length, v[2] / length];
      } else {
        return [0, 0, 0];
      }
    }

下面是计算一个 "lookAt" 矩阵的代码。

    function makeLookAt(cameraPosition, target, up) {
      var zAxis = normalize(
          subtractVectors(cameraPosition, target));
      var xAxis = cross(up, zAxis);
      var yAxis = cross(zAxis, xAxis);
     
      return [
         xAxis[0], xAxis[1], xAxis[2], 0,
         yAxis[0], yAxis[1], yAxis[2], 0,
         zAxis[0], zAxis[1], zAxis[2], 0,
         cameraPosition[0],
         cameraPosition[1],
         cameraPosition[2],
         1];
    }

这是我们如何使用它来使相机随着我们移动它指向在一个特定的 ‘F’ 的。

      ...
     
      // Compute the position of the first F
      var fPosition = [radius, 0, 0];
     
      // Use matrix math to compute a position on the circle.
      var cameraMatrix = makeTranslation(0, 50, radius * 1.5);
      cameraMatrix = matrixMultiply(
          cameraMatrix, makeYRotation(cameraAngleRadians));
     
      // Get the camera's postion from the matrix we computed
      cameraPosition = [
          cameraMatrix[12],
          cameraMatrix[13],
          cameraMatrix[14]];
     
      var up = [0, 1, 0];
     
      // Compute the camera's matrix using look at.
      var cameraMatrix = makeLookAt(cameraPosition, fPosition, up);
     
      // Make a view matrix from the camera matrix.
      var viewMatrix = makeInverse(cameraMatrix);
     
      ...

下面是结果。

<p align="center">
<a class="webgl_center" target="_blank" href="../webgl-3d-camera-look-at.html">点击这里在新窗口中打开</a>
<p>

拖动滑块，注意到相机追踪一个 ‘F’。

请注意，您可以不只对摄像机使用 “lookAt” 函数。共同的用途是使一个人物的头跟着某人。使小塔瞄准一个目标。使对象遵循一个路径。你计算目标的路径。然后你计算出目标在未来几分钟在路径的什么地方。把这两个值放进你的 lookAt 函数，你会得到一个矩阵，使你的对象跟着路径并且朝向路径。

让我们<a href="webgl-animation.html">接下来了解下动画</a>。

### lookAt 标准

大多数 3D 数学库有一下 lookAt 函数。通常它是被专门设计制用来生成一个“视图矩阵”，而不是一个“相机矩阵”。换句话说，它产生了一个使其他一切物体移动到相机的前面的矩阵，而不是一个移动相机本身的矩阵。

我觉得那不太有用。像上面说的，lookAt 函数有很多用途。当你需要一个视图矩阵时很容易调用逆矩阵，但如果您正使用 lookAt 函数使一些人物的头跟随其他人，或者使塔指向它的目标，如果 lookAt 函数返回一个矩阵，它可以定位世界空间中的物体，在我看来这会更有用。

<p align="center" >
<a class="webgl_center" target="_blank" href="../webgl-3d-camera-look-at-heads.html">点击这里在新窗口中打开</a>
</p>

问题？[可以在 stackoverflow 提问](http://stackoverflow.com/questions/tagged/webgl)。

问题/缺陷？[可以在 github 创造一个话题](https://github.com/greggman/webgl-fundamentals/issues)。