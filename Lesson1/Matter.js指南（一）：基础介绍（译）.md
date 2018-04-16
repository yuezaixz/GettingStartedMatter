# Matter.js指南（一）：基础介绍（译）

本文是系列文章 [Getting Started With Matter.js](https://code.tutsplus.com/series/getting-started-with-matterjs--cms-1186) 的第一篇。

Matter.js是用JavaScript编写的2D实体物理引擎。这个库可以帮助你轻易的在浏览器中模拟2D物理环境。它提供了一些功能，比如创建实体形状并赋予他物理属性（如质量，面积或密度）。你可以模拟出多种多样的力和碰撞情况（比如重力和摩擦力）。


## 原文

点击下面链接可以访问原文：
[Getting Started With Matter.js: Introduction](https://code.tutsplus.com/tutorials/getting-started-with-matterjs-introduction--cms-28784)

## 源码

该系列文章源码地址：
[GettingStartedMatter](https://github.com/yuezaixz/GettingStartedMatter)


## 引入

可以通过Bower或者NPM这些包管理器来添加

```
bower install matter-js
npm install matter-js
```

也可以通过[CDN](https://cdnjs.com/libraries/matter-js)或者本地文件路径来添加。

```
<script src="path/to/matter.min.js"></script>
```

## 最基本的例子

学习Matter.js最好的方式就是去看例子的代码，并且去理解它的工作原理。在这个篇章中，我们将通过代码创建一些实体并一行一行来理解代码。

``` html

<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Getting Started With Matter.js: Introduction</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/matter-js/0.12.0/matter.js"></script>
    <style>
        body {
            margin: 20px auto;
            text-align: center;
        }
    </style>
</head>
<body>
    <script>
        var Engine = Matter.Engine,
        Render = Matter.Render,
        World = Matter.World,
        Bodies = Matter.Bodies;

        var engine = Engine.create();

        var render = Render.create({
            element: document.body,
            engine: engine,
            options: {
                width: 800,
                height: 400,
                wireframes: false
            }
        });

        var boxA = Bodies.rectangle(400, 200, 80, 80);
        var ballA = Bodies.circle(380, 100, 40, 10);
        var ballB = Bodies.circle(460, 10, 40, 10);
        var ground = Bodies.rectangle(400, 380, 810, 60, { isStatic: true });

        World.add(engine.world, [boxA, ballA, ballB, ground]);

        Engine.run(engine);

        Render.run(render);
    </script>
</body>
</html>

```

我们首先为我们项目中可能需要的所有Matter.js模块创建局部变量，Engine模块包含创建和控制引擎的方法。一个项目需要引擎来更新世界的模拟。Render模块是基于canvas的基础渲染器，该模块被需要用在可视化不同引擎上。

```World```模块用于创建和控制引擎所运行的世界。它类似与```Composite```模块，但它可以让您调整例如重力和边界这些额外的属性。最后还有一个模块称为```Bodies```，你可以用他来创建实体对象，有一个与之名字非常接近的模块叫```Body```，是用来控制单个实体，要记住他们之间是不一样的，一个是用来创建实体，一个是用来控制实体。

接下来我们使用```Engine```模块的```create([settings])```函数来创建一个新的引擎，其中settings参数实际上是一个键值对的Map对象，用来更改引擎的一些属性。

例如，可以控制World中所有实体的时间因素。值小于1时将导致World中的实体相互作用变的缓慢。反之，大于1将加快World的运转。在本系列后续教程中可以了解更多关于Matter.Engine模块的知识。

然后，我们使用```Render```模块的```create([settings])```函数来创建一个新的渲染器。就像```Engine```模块一样，上述方法中的settings参数是一个键值对的Map对象，可以根据需要设置一些参数选项。比如可以使用```element```指定需要在其中插入画布；或者，你也可以使用```canvas```来指定在哪里进行Matter.js世界的渲染。

划重点，```engine```可以用来指定用来渲染的引擎。还有一个```options```参数，值是一个对象。 可以使用这个值对不同的参数进行设置，例如画布的宽度或高度。 您也可以通过分别将```wireframes```key的value设置为true或false来打开或关闭线框。

接下来的几行在```World```中创建了不同的实体，这些实体是通过Matter.Bodies函数来创建的。在这个例子中，我们刚刚使用```circle()```和```rectangle()```方法创建了两个圆和一个矩形。 当然还有一些其他函数可用于创建不同的多边形。

刚刚我们是创建了实体，但还需要使用```World```模块中的```add()```函数将它们添加到我们选定的世界中，但这还不够，在添加后，我们还需要调用```Engine```和```Render```模块的```run()```函数来将引擎和渲染器运行起来。到这里差不多了，这就是在Matter.js中创建和渲染世界的最简单而有最基本的代码段了。

在👇你可以看到代码的运行效果。

<iframe src="https://codepen.io/Shokeen/embed/JNZYVP/?height=500&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="580" frameborder="0" scrolling="no"></iframe>

## Matter.js的基础模块

在```Matter```有多达20多个不同的模块，这些模块又提供了多种函数和属性来有效的创建各种模拟环境并让你可以与他们交互。他们有的用来处理碰撞，有的用来处理渲染和模拟。

上一节中的示例使用了四个不同的模块来处理渲染、模拟和实体创建。在本节中，您将了解Matter.js中一些常用模块的作用。

* Engine：引擎是用来不断更新Matter.js所模拟的世界。Engine模块提供了很多函数和属性，允许你来控制引擎的各种行为
* World：该模块为您提供即时的创建和控制整个世界的函数和属性。```World```实际上是一个具有像重力和边界等附加属性的```Composite```
* Bodies: Bodies模块包含各种函数来帮助您创建具有常见形状（如圆形、矩形或梯形）的实体
* Body: 该模块为您提供了各种的方法和属性，用于创建和操作使用Bodies模块中的函数创建的实体。该模块允许您缩放、旋转或移动单个物体。 它提供函数来改变物体的速度、密度或惯性。因为改模块所具有的能力颇多，我们将在本系列的第三篇教程中深入的讨论
* Composites: 类似```Bodies```模块，该模块也有很多的函数让你可以使用这些函数来创建具有通用配置的复合实体。例如，你可以通过```Composites```模块的特定函数创建通过矩形组成的堆或金字塔
* Composite: ```Composite```模块提供函数和属性让你可以轻易的创建和控制复合实体。我们将在本系列的第四个教程详细的了解复合实体。
* Constraint: 该模块允许你创建和控制约束。你可以使用约束来限制两个物体或固定的点与物体间保持在固定的距离。可以想象它通过钢杆连接两个物体。还可以修改这些约束的刚度，使杆开始更像弹簧。当需要创建牛顿摇篮或链式组合物体时，我们可以使用Matter.js的```Constraint```模块才实现。
* MouseConstraint：该模块提供函数和属性来创建和控制鼠标约束，这在想让用户与```World```中的物体交互时候特别有用。

## 思考

这篇课程主要是向你介绍```Matter.js```库，记住我们是怎么快速的安装该库并用该库创建两个圆一个矩形盒子的虚拟场景。然后```Matter.js```有哪些模块，他们有哪些不一样的函数去影响我们的引擎。本文已经讲到了这些模块，在接下来的文章中我们将更深入的了解他们。

