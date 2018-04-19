# Matter.js指南（三）：Body模块（译）

本文是系列文章 [Getting Started With Matter.js](https://code.tutsplus.com/series/getting-started-with-matterjs--cms-1186) 的第二篇。

上篇中，我们简短的介绍了Matter.js的World和Engine模块。通过这两个模块的函数和属性，我们可以控制整个虚拟世界的特性。但是，有时候我们只想控制其中几个实体的特性。

比如，我想施加一些力在一个实体上，或者我想改变它的摩擦力。Matter.js的Body模块这时候就闪亮登场了，他可以实现这些案例。这个模块有很多函数和属性让你可以随意操作实体的物理特征，不管是质量，还是弹力，都可以。那本篇文章就让让我们来学习Body模块吧。


## 原文

点击下面链接可以访问原文：
[Getting Started With Matter.js: The Body Module](https://code.tutsplus.com/tutorials/getting-started-with-matterjs-body-module--cms-28835)

## 源码

该系列文章源码地址：
[GettingStartedMatter](https://github.com/yuezaixz/GettingStartedMatter)

## 缩放、旋转、跳跃，不停歇

* rotate(body, rotation)函数可以旋转实体，它会从实体当前的角度开始旋转参数中指定的弧度，但是不会给实体赋予任何速度。
* scale(body, scaleX, scaleY, [point])函数可以用来缩放物体，scaleX和scaleY指定了水平方向和垂直方向的缩放比例。但记住缩放会改变物体的质量、面积及惯性这些物理特征。第四个参数是指定缩放发生的点，默认为实体的中心点。
* translate(body, translation)函数可以让物体沿着一个向量进行一定的位移，translation参数就是这个向量，移动后的坐标就是起始坐标通过这个向量位移后的结果。

下面是一个示例，可以结合代码和效果体验下这3个函数及其参数带来的效果。

```

var Body = Matter.Body;
var box = Bodies.rectangle(460, 120, 40, 40);
 
$('.scale').on('click', function () {
    Body.scale( box, 1.5, 1.2);
});
 
$('.rotate').on('click', function () {
    Body.rotate( box, Math.PI/6);
});
 
$('.translate').on('click', function () {
    Body.translate( box, {x: -10, y: 20});
});

```

<iframe src="https://codepen.io/Shokeen/embed/EmpMgo/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

## 速度与受力

* setVelocity(body, velocity)函数可以改变实体的线性速度，他不会改变物体的角度，也不会对物体施加力或改变位置，可能物体会因为摩擦力等因素改变角度、位置，但显然这不是因为该函数的作用。注意，参数velocity是一个向量
* setAngularVelocity(body, velocity)可以改变实体的角速度，同样，该函数不会直接改变位置、角度。注意，参数velocity是一个数值

```

$('.linear').on('click', function () {
    Body.setVelocity( box, {x: 10, y: -10});
});
 
$('.angular').on('click', function () {
    Body.setAngularVelocity( box, Math.PI/6);
});

```

<iframe src="https://codepen.io/Shokeen/embed/XRBGMz/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

* applyForce(body, position, force)：除了改变物体的速度与角速度，还可以直接施加力在实体上，force参数是一个向量，代表里的方向及大小，position参数代表力施加的位置。该函数可能为让物体产生扭矩（也就是旋转）

下面代码中，通过参数{x: 0, y: -0.05}、{x: 0.05, y: 0}施加了垂直方向和水平方向的力，施力点都是body.position，也就是该物体的中心位置，这样就不会产生扭矩了。注意垂直朝上的力是负值的，同理水平向左的力也是负值的，这点注意下。还有一点注意下，因为重力的作用，物体一直受到一个垂直向下的力，即重力加速度1。

<iframe src="https://codepen.io/Shokeen/embed/WjKmMG/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

只要该球不与任何墙壁或地板碰撞，施加力后的球运动似乎是自然的。通常情况下，当事物与事物相撞时，我们期望它们反弹。物体的弹力由还原系数决定。

在Matter.js中，还原系数默认为0，也就是没有任何弹力。如果设置成1，那么碰撞后不会有任何弹力势能流失，而设置成0.5，那么碰撞后弹性势能将丢失一半。可以通过restitution键值来设定还原系数。

另外，在虚拟世界中，动态摩擦力、空气阻力、静态摩擦力也是非常需要的。

* friction键值为动态摩擦力，可以设置为0到1之间的值，0代表物体表面绝对光滑，那么想要物体停下来只能施加而外的力。两个物体之间的动态摩擦力是根据公式Math.min(bodyA.friction, bodyB.friction)确定的
* frictionStatic定义的是物体的静态摩擦力，默认为0.5，值越代表物体从静止到运动所需要的力越大
* frictionAir定义的是物体的空气阻力，也就是物体与空气的摩擦力。值越大代表物体的空气中速度降低的速度越快。注意，空气阻力的影响并不是线性的。

```

$('.red-friction').on('click', function () {
  circleA.friction = 0.05;
  circleA.frictionAir = 0.0005;
  circleA.restitution = 0.9;
});
 
$('.res-friction').on('click', function () {
  circleA.friction = 0.1;
  circleA.frictionAir = 0.001;
  circleA.restitution = 0;
});

```

<iframe src="https://codepen.io/Shokeen/embed/BRPEJw/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

## 渲染

到目前为止，我们还未指定渲染时的填充色、边框及边框样式。所有这些属性都嵌套在render这个键值中。fillStyle属性接受一个字符串来指定实体的填充样式。lineWidth属性接受一个数字，该数字定义创建实体时的边框，设置为0代表无边框。strokeStyle属性为边框的样式。

你也可以通过设置visible为false来让物体不需要渲染。也可以通过opacity来改变物体的透明度。

也可以通过图片来代替这些样式，如以下代码

```
var ball = Bodies.circle(90, 280, 20, {
  render: {
    sprite: {
      texture: "path/to/soccer_ball.png",
      xScale: 0.4,
      yScale: 0.4
    }
  }
});
```

xOffset和yOffset属性可用于定义sprite在各个轴的偏移量。同样，您可以使用xScale和yScale属性为sprite定义x轴和y轴的缩放比例。可以通过[soccer sprite from the Open Game Art](https://opengameart.org/content/soccer-ball) 来获取图片。

<iframe src="https://codepen.io/Shokeen/embed/QvZYjm/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

## 改变物理特征

我们已经了解过摩擦力、弹力这些物理特征了。还有一些属性也可以同样的方式去设定。但也还有一些属性是只读的，你不能直接改变他们。

比如你可以通过position属性来设定位置，你也可以通过mass属性来设置质量，但通过density属性来控制物体的质量是更好的方式。

一旦你改变了密度，其质量将根据其面积自动计算。通过这种方式，您还可以根据密度来区分不同的对象。例如，使用岩石作为sprite应该比使用足球作为sprite具有更高的密度。

速度和角速度等属性是只读的，但它们的值可以使用setAngularVelocity()和setVelocity()等适当的方法来设置。

可以通过[文档](http://brm.io/matter-js/docs/classes/Body.html#properties)来更深入的学习。

## 总结

在本教程中，您已经了解了Matter.js的Body模块一些重要函数和属性。 了解这些不同的属性以及它们的作用可以帮你创建更真实的虚拟世界。在本系列的最后一篇教程中，您将学习Matter.js中的复合模块。



