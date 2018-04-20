# Matter.js指南（四）：Composites与Composite 模块（译）

本文是系列文章 [Getting Started With Matter.js](https://code.tutsplus.com/series/getting-started-with-matterjs--cms-1186) 的第四篇。

上篇中，我们详细的学习了Body模块，知道了如何操作像圆、矩形、梯形这类图形。但那些简单都比较简单，本章节我们将学习创建、操控如车、链式结构、金字塔、堆，以及布状结构这样的物体。

我们可以通过Composites模块创建他们，然后，通过Composite模块的函数和属性来控制他们。


## 原文

点击下面链接可以访问原文：
[Getting Started With Matter.js: The Composites and Composite Modules](https://code.tutsplus.com/tutorials/getting-started-with-matterjs-composite-and-composites-module--cms-28836)

## 源码

该系列文章源码地址：
[GettingStartedMatter](https://github.com/yuezaixz/GettingStartedMatter/tree/master/Lesson4)

## 创建堆与金字塔

堆和金字塔其实是非常相近的，可以通过stack(xx, yy, columns, rows, columnGap, rowGap, callback)来创建堆，类似的，通过pyramid(xx, yy, columns, rows, columnGap, rowGap, callback)函数可以创建金字塔。他们的参数都是一样的，实际上，金字塔形成是从堆的形状中得到的。

参数已经说明的很清楚了，xx与yy就是实体的起始点，columns与rows参数决定了列数与行数，columnGap与rowGap决定了列间距与行间距。

在重力的影响下，行间距很快就会消失，而由于重力产生的动量可能会改变复合实体的形状。

回调函数用于构建每一个复合实体中的一个一个小格。这意味着您可以使用它来创建矩形框或梯形的堆栈或金字塔。您应该记住，使用圆圈会使结构不稳定。这里是创建一堆矩形的代码：


```
var stack = Composites.stack(550, 100, 5, 3, 0, 0, function(x, y) {
  return Bodies.rectangle(x, y, 40, 20, {
    render: {
      fillStyle: 'orange',
      strokeStyle: 'black'
    }
  });
});
```

你可以让回调函数很复杂来创建更有趣的复合实体，但本例中我们只用了上一节学到的矩形创建方法，让单元格以橙色填充黑色边框来展现。

<iframe src="https://codepen.io/Shokeen/embed/YVRXbx/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" scrolling="no"></iframe>

创建金字塔的话也很简单，像这样就可以了

```
var pyramid = Composites.pyramid(0, 220, 11, 6, 0, 0, function(x, y) {
  return Bodies.rectangle(x, y, 30, 30, {
    render: {
      fillStyle: 'cornflowerblue',
      strokeStyle: 'black'
    }
  });
});
```

当你创建金字塔时候，有时候你会发现创建的行数少于你指定的行数，那是因为函数有隐性的规定，可以参考这个公式

```
Math.min(rows, Math.ceil(columns / 2))
```

<iframe src="https://codepen.io/Shokeen/embed/LyXmga/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" scrolling="no"></iframe>

你可以小心翼翼的摆放一个堆或一个金字塔放在一个金字塔上以创建有趣的图案。例如，你可以在红色的金字塔上放置一个较小的金字塔以创建一个具有两种颜色的完整金字塔。

## 创建车与链式结构

车在Matter.js中是两个轮子与一个车身所组成的形状，轮子的摩擦力为0.8，质量为0.01，可以通过car(xx, yy, width, height, wheelSize)创建一辆车，xx与yy用于指定车的坐标位置。

weight与height决定了车身的坐标，wheelSize参数决定了轮子的半径，创建车与堆和金字塔不一样，他不需要回调函数。

```
var car = Composites.car(190, 100, 100, 45, 30);
 
$('.force').on('click', function () {
    Body.applyForce( car.bodies[0], {x: car.bodies[0].position.x, y: car.bodies[0].position.y}, {x: 0.5, y: 0});
});
```

通过chain(composite, xOffsetA, yOffsetA, xOffsetB, yOffsetB, options)可以创建链式复合实体，其实就是用约束将盒子们捆绑在一起，几个offset参数用于确定连接不同盒子约束的相对位置。

我们需要一个额外的约束让链式复合实体悬挂在虚拟世界的某个点上，如如下代码：

```
var boxes = Composites.stack(500, 80, 3, 1, 10, 0, function(x, y) {
        return Bodies.rectangle(x, y, 50, 40);
    });
 
var chain = Composites.chain(boxes, 0.5, 0, -0.5, 0, { stiffness: 1});
 
Composite.add(boxes, Constraint.create({ 
        bodyA: boxes.bodies[0],
        pointB: { x: 500, y: 15 },
        stiffness: 0.8
    }));
```

链式复合实体中的框已经使用前面学到的stack()函数创建了。然后chain()函数创建的约束具有1的韧性（也就是没弹性，长度不会变化）的链条。通过额外的约束我们让他挂在天花板上。

这是一个汽车和链式复合实体的粒子。你可以通过橙色按钮让车前进后退。每次点击都会在第一个轮子中心施加一个力，让他动起来。

<iframe src="https://codepen.io/Shokeen/embed/rmQOzM/?height=650&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="700" frameborder="no" scrolling="no"></iframe>


## 创建柔性实体及牛顿摆

柔性实体类似于堆，但有两点较大的差异。一是柔性实体中的每个单元格会通过约束悬挂着相邻的元素，二是柔性实体可以用圆作为构成元素。您可以将柔软的身体视为网格和堆栈之间的交叉。通过调用softBody(xx, yy, columns, rows, colGap, rowGap, crossBrace, pRadius, pOptions, cOptions)函数来创建柔性实体。前6个参数已经非常熟悉了，crossBrace参数是一个布尔值，用于确定是否应该显示交叉的部分。pRadius参数确定圆的半径，pOptions参数可用于控制粒子的其他属性，如质量和惯性。

cOptions参数指定将粒子绑定在一起的约束条件的各种选项。以下代码将为我们的Matter.js世界创建一个柔体。

```
var particleOptions = { 
        friction: 0.05,
        frictionStatic: 0.1,
        render: { visible: true } 
    };
 
var constraintOptions = { 
        render: { visible: false } 
    };
 
var softBody = Composites.softBody(450, 200, 10, 5, 0, 0, true, 15, particleOptions, constraintOptions);
```

<iframe src="https://codepen.io/Shokeen/embed/EmOLJO/?height=650&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="700" frameborder="no" scrolling="no"></iframe>

通过newtonsCradle(xx, yy, number, size, length)函数可以创建牛顿摆，number决定摆球的数量，size决定每个摆球的半径，length决定悬挂的线索的长度，Matter.js库将还原和摩擦值设置为零，以便他们可以长时间继续运动。

下面的代码创建牛顿摆并将第一个球移动到更高的位置，以便当球落下并撞击其他球时它有一定的速度。translate()函数指定的位置相对于主体的当前位置。所有这些功能和Body模块的属性在本系列的前一课程中已经有详细的介绍了。

```
var cradleA = Composites.newtonsCradle(200, 50, 5, 20, 250);
Body.translate(cradleA.bodies[0], { x: -100, y: -100 });
```

<iframe src="https://codepen.io/Shokeen/embed/eWQyLM/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" scrolling="no"></iframe>

## Composite模块中重要的函数和属性

我们已经学习了怎么创建这些复合实体，是时候学习下另外一些Composite中重要的函数和属性了。比如我们可以通过rotate(composite, rotation, point, [recursive=true])、scale(composite, scaleX, scaleY, point, [recursive=true])与translate(composite, translation, [recursive=true])来旋转、缩放以及移动实体。其实这些与Body模块都非常相似。

你也可以使用add(composite，object)和remove(composite，object，[deep = false])函数分别添加或删除给定复合实体中的一个或多个元素、约束以及复合实体。如果你想将一些物体从一个复合实体移动到另一个复合实体，可以借助move(compositeA，objects，compositeB)函数来完成。该函数将把给定的对象从compositeA移动到compositeB。

如果你想要获得所有组合体的直接子对象，组合体和约束条件，则可以使用composite.bodies，composite.composites和composite.constraints属性。

我们已经看到如何使用bodies属性将牛顿摆的球移动到左侧，并在汽车的车轮上施加一个力。一旦你对复合实体中的各个实体进行了引用，就可以使用Body模块的所有方法和属性来控制它们。

## 最后，思考下

在本教程中，我们学习了如何使用Matter.js中的Composite和Composites模块创建一些复杂的复合实体。还了解了可用于控制这些复合材料的不同函数和属性。

本系列旨在让初学Matter.js库的人以适合初学者的方式入门。牢记这一点，我们已经涵盖了库中最常见模块的重要功能和属性。

Matter.js还有很多其他模块，我们在本系列的第一个教程中对此进行了简要讨论。如果你想充分利用这个库，你应该阅读官方网站上所有这些模块的[文档](http://brm.io/matter-js/docs/index.html)。

