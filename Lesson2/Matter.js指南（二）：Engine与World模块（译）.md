# Matter.js指南（二）：Engine与World模块（译）

本文是系列文章 [Getting Started With Matter.js](https://code.tutsplus.com/series/getting-started-with-matterjs--cms-1186) 的第二篇。

上篇中，我们简短的介绍了很多Matter.js的模块。这个库模块很多，一个篇章想说清楚太不现实，整个篇章也只是介绍了一些简单的使用。不过看完第一篇章，你应该也对整个库的框架有了一定的了解了吧。

本篇中，我们将聚焦在Engin与World模块。World模块提供了创建和控制虚拟世界所必须的函数和属性，通过他们我们可以在世界中创建、移除实体。而Engine模块可以创建和控制对虚拟世界运行至关重要的运行引擎。


## 原文

点击下面链接可以访问原文：
[Getting Started With Matter.js: Introduction](https://code.tutsplus.com/tutorials/getting-started-with-matterjs-introduction--cms-28784)

## 源码

该系列文章源码地址：
[GettingStartedMatter](https://github.com/yuezaixz/GettingStartedMatter)

## World模块

这个章节中，你可以学到World模块的属性、函数及事件。查看源码就可以发现，World实际就是继承自Composite模块，但他添加了类似重力、弹力这些额外的属性。然后还增加了重要的方法，可以看下面代码及注释

```

export class World extends Composite {
				/*
				** 这个函数重写自Composite，它可以添加实体、组件或约束到虚拟世界中
				*/
        static add(world: World, body: Body | Array<Body> | Composite | Array<Composite> | Constraint | Array<Constraint> | MouseConstraint): World;
				/*
				** 该函数可以添加单个实体到虚拟世界中
				*/
        static addBody(world: World, body: Body): World;
				/*
				** 该函数可以添加单个组件到虚拟世界中
				*/
        static addComposite(world: World, composite: Composite): World;
				/*
				** 该函数可以添加单个约束到虚拟世界中
				*/
        static addConstraint(world: World, constraint: Constraint): World;
        static clear(world: World, keepStatic: boolean): void;
        static create(options: IWorldDefinition): World;

        gravity: Gravity;
        bounds: Bounds;

    }
    
```

我们再来看看下面代码，这里我们添加了多个实体到虚拟世界中。add()函数添加3个静态的矩形作为墙；addBody()函数添加一个通过点击产生的圆作为球。

```
var topWall = Bodies.rectangle(400, 0, 810, 30, { isStatic: true });
var leftWall = Bodies.rectangle(0, 200, 30, 420, { isStatic: true });
var ball = Bodies.circle(460, 10, 40, 10);
var bottomWall = Bodies.rectangle(400, 400, 810, 30, { isStatic: true });
 
World.add(engine.world, [topWall, leftWall, ball, bottomWall]);
 
 
var addCircle = function () {
 return Bodies.circle(Math.random()*400 + 30, 30, 30);
};
 
$('.add-circle').on('click', function () {
    World.addBody(engine.world, addCircle());
});
```

可以看到作为墙的三个实体的isStatic属性设置为true，这么做是为了把这三个实体设置成完全静止的物体，他永远不会移动及旋转，而墙就需要这么个特性。这些神奇的属性，我们将在第三篇中Body章节深入学习，这里不做过多介绍，我们先关注World模块。

<iframe src="https://codepen.io/Shokeen/embed/LyrLZq/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

* remove( composite, object, [deep=false]): 这个函数可以从虚拟世界中移除一个或多个实体、组件或约束。该函数继承自Composite
* clear( world, keepStatic): 该函数是继承自Composite中函数的别名。他可以清楚虚拟世界中的所有元素。keepStatic参数默认为false，即删除所有元素包括静态的，如果设置为true，那么isStatic为true的元素将不会被移除。
* rotate( composite, rotation, point, [recursive=true]):该函数可以以一个设定的角度以point为中心旋转组件及其包含的所有元素，rotation参数在这里是弧度。
* scale( composite, scaleX, scaleY, point, [recursive=true]): 该函数可以缩放组件及其包含的所有元素，他不仅缩放长度、宽度、面积，还会缩放质量、惯性。
* translate(composite, translation, [recursive=true]): 这个函数可以让整个组件从当前点，沿着translation移动，translation是一个向量(x0,y0)，也就是x轴上移动x0，y轴上移动y0.


请记住，translate()与rotate()都不会改变世界中实体的速度。发生的运动是因为位置和形状发生变化，以下是旋转、移动、缩放的例子。

```

$('.scale').on('click', function () {
    Matter.Composite.scale( engine.world, 0.5, 0.7, {x: 400, y: 200});
});
 
$('.rotate').on('click', function () {
    Matter.Composite.rotate( engine.world, Math.PI/4, {x: 400, y: 200});
});
 
$('.translate').on('click', function () {
    Matter.Composite.translate( engine.world, {x: 10, y: 10});
});

```

您应该注意，上面的代码在x和y轴上应用了不同的比例。这将导致Matter.js世界中的圆圈变成椭圆形。椭圆随后倾向于以更低的势能进入更稳定的状态。

<iframe src="https://codepen.io/Shokeen/embed/NjzaoG/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

点击scale按钮，发现圆变椭圆之后，按下rotate按钮，Matter.js的虚拟世界就倾斜了。

除了这些函数之外，World模块还有很多有用的属性。 可以使用world.bodies获得组件中的所有body集合。world.composites和world.constraints同理。

还可以通过world.bounds定义虚拟世界的碰撞范围。更有趣的是还可以改变这个世界的重力，而且可以分别设置x轴及y轴上的重力。

你可以通过[Matter.World documentation page](http://brm.io/matter-js/docs/classes/World.html)文档了解的更深入，甚至你可以去阅读Matter.js的源码。

## Engine模块

Engine模块是正确更新模拟世界所必需的。以下是Engine模块的一些重要方法：

* create([options]): 这个方法用于创建引擎，options参数实际上是键值对，通过这个参数可以覆盖引擎中很多默认的设置，例如设置timeScale属性可以改变虚拟世界的时间快慢
* update(engine, [delta=16.666], [correction=1]): 该函数用于设置引擎的刷新率，也就是两次update之间的时间间隔。correction是修正系数，我理解就是delta变化的百分比，即delta / lastDelta。
* merge(engineA, engineB): 用engineA的配置作用于engineB的world，合并为一个新的引擎。

引擎模块还具有许多其他属性以帮助你控制模拟世界的质量。比如constraintIterations、positionIterations或velocityIterations，可以执行更新期间要执行的约束、定位及速度迭代的次数。值越大模拟的效果越好，但设备性能要求也越高。

我们可以试试用timing.timeScale来改变虚拟时间的时间，如下面这个例子中三个按钮，让时间变慢，变快及恢复正常。

```
$('.slow-mo').on('click', function () {
    engine.timing.timeScale = 0.5;
});
 
$('.norm-mo').on('click', function () {
    engine.timing.timeScale = 1;
});
 
$('.fast-mo').on('click', function () {
    engine.timing.timeScale = 1.5;
});
```

<iframe src="https://codepen.io/Shokeen/embed/oWMNoO/?height=550&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" width="850" height="600" frameborder="no" allowfullscreen="true" scrolling="no"></iframe>

通过点击按钮可以感觉到时间快慢上的变化。

这里我们还用到了弹力，这些我们将在下一篇章中学到。对这一章节中有什么不了解的，可以通过[Matter.Engine documentation page](http://brm.io/matter-js/docs/classes/Engine.html) 来学习。

## 总结

本篇讨论了Matter.js中两个非常重要的模块，为了让你的虚拟世界运行起来，你必须学会他们。现在，你应该能够缩放、旋转、放慢或加快你的世界。 还有，你也知道如何移除或添加一个世界的实体。这在开发2D游戏时是至关重要的。

在本系列的下一个教程中，您将了解Bodies模块中可用的不同函数、属性和事件。



