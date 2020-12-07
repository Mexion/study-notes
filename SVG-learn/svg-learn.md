 ## 基本图形

- <rect> (矩形)
- <circle> (圆形)
- <ellipse> (椭圆)
- <line> (直线)
- <polyline> (折线)
- <polygon> (多边形)

没有曲线单独的标签，曲线一般用<path>进行绘制，<path>可以绘制任何图形，上面的任何图形都可以用<path>进行绘制，上面定义的标签只是为了方便绘制而已。



## 基本属性 

- fill (填充颜色)

- stroke (绘制颜色)

- stroke-width (绘制宽度)

- transform (图形变换)

所有的图形都会用到这些属性。

- <rect>属性有`x`,`y`,`width`,`height`,`rx`,`ry`。`x`,`y`为图形的位置(左上角)，`width`为宽，`height`为高，`rx`，`ry`为两个方向的圆角半径值。

- <circle>属性有`cx`,`cy`,`r`。`cx`,`cy`表示圆心的相对位置，`r`表示半径。

- <ellipse>属性有`cx`,`cy`,`rx`,`ry`。`cx`,`cy`表示圆心的相对位置，`rx`,`ry`表示圆角半径。

- <line>的属性有`x1`,`y1`,`x2`,`y2`。分别表示两个端点的位置。

- <polyline>的属性有`points`，表示绘制点，型如`points="x1 y1 x2 y2 x3 y3"`，之间用空格或者逗号隔开。

- <polygon>和折线相同，只不过会自动将起点和终点闭合。



## 创建标签

在`Html`中直接书写`svg`标签，指定`xmlns`即可。

`JS`创建`svg`用`document.createElementNS(ns,targetName)`，`ns`为`http://www.w3.org/2000/svg`。
插入用`appendChild(Element)`，设置获取属性用`setAttribute(key, value)`，`getValue(key)`。



## 世界，视窗和视野

### viewBox 

`SVG`中世界是无穷大的，而视窗（`viewport`，就是`SVG`标签`width`和`height`指定的区域）是浏览器绘制的一个区域，通过`css`或者`width`和`height`控制大小。每一个`SVG`标签都是一个无穷大的世界，而我们指定的`width`和`height`是我们指定的显示区域。那么我们怎么控制这个显示区域显示这个世界的哪个位置呢？`viewbox`属性就是用来干这个的，`viewbox`可以指定一块区域在视窗中进行显示。当`viewbox`不指定时，默认为`0 0 width height`。

```html
<svg xmlns="http://www.w3.org/2000/svg"
width="800" height="600" viewbox="0 0 400 300">
</svg>
```

`viewBox`值有4个值：`viewBox="x y width height"`，`x`为相对于左上角的横坐标，`y`为相对于左上角的纵坐标，`width`为宽度，`height`为高度。

> 当 `viewBox `的宽高小于视窗的宽高时，相当于放大。  当 `viewBox `的宽高大于视窗的宽高时，相当于缩小。 

怎么理解这个缩放的概念呢？

网上很多教程讲的乱七八糟，一顿比喻实际上把人搞得晕头转向，我这里举一个我自己想到的例子，绝对简单易懂。

我们拿现实世界来打比方。一个`SVG`画布类比为我们的现实世界，它是无限大的；我们在`SVG`标签上指定的`width`和`height`类比为我们的电脑显示器大小；`viewbox`相当于一部可以拍摄无限大小区域的照相机。我们可以使用`viewbox`到世界的任何地方去拍摄任意大小的内容，但是拍摄的内容要放到我们的电脑显示器上显示。假如我们拍摄了一个`1000 * 1000`大小的区域，如果按照原比例进行显示，那么我们也需要一个`1000 * 1000`大小的电脑显示器才能将这片区域完全显示出来。但是我们此时只有一个`100 * 100`大小的显示器（此时`SVG`的`width`和`height`都是`100`，而`viewbox`大小为`1000 * 1000`），要完全显示只能将这张照片进行缩放显示，那么很显然，缩小为`1 / 10`即可。同理，假如我们只拍摄了一个`10 * 10`的区域，要放到`100 * 100`的显示器上，就要放大为原来的`10`倍即可。

现实世界中，当我们将照片全屏显示在屏幕上时，照片会自动缩放成和屏幕一个大小，在`SVG`中，自然也是如此，`SVG`的缩放不需要我们手动进行，`viewbox`截取的区域会自动缩放到视窗中。

### preserveAspectRatio

`preserveAspectRatio `属性用来保持图形的宽高比。

如果 `viewBox `的宽高比和 视窗的宽高比不同，那么在拉伸` viewBox `来适应视窗的时候，就可能导致 `SVG `图形发生扭曲。控制`preserveAspectRatio` 来指定宽高比（默认值`xMidYMid meet`）。

```html
preserveAspectRatio = <align> <meetOrSlice>?
```

**align**

`align` 的单个值如下:

| 值   | 含义                                        |
| ---- | ------------------------------------------- |
| none | 通过拉伸 viewBox 来适应整个视窗，不管宽高比 |
| xMin | viewBox 和 viewport 左边缘对齐              |
| xMid | viewBox 和 viewport x 轴中心对齐            |
| xMax | viewBox 和 viewport 右边缘对齐              |
| YMin | viewBox 和 viewport 上边缘对齐              |
| YMid | viewBox 和 viewport y 轴中心对齐            |
| YMax | viewBox 和 viewport 下边缘对齐              |

组合 `xy` 可以结合为一个`align`参数，如：

```
xMinYMin => 左-上对齐
xMidYmid => 中-中对齐
```

**meetOrSlice**

| 值    | 含义                                                         |
| ----- | ------------------------------------------------------------ |
| meet  | 保持宽高比缩放 viewBox 以适应 viewport，类似于 `background-size: contain` |
| slice | 保持宽高比同时比例小的方向放大填满 viewport，类似于 `background-size: cover` |

## 分组

`SVG`复杂的图形不是由一个标签就可以生成的，而是多个标签组合在一起的，如果我们要整体移动某一个复杂图形，改变整个图形的颜色，那么一个一个移动修改显然不现实，这时候就需要分组。

> 元素`g`是用来组合对象的容器。添加到`g`元素上的变换会应用到其所有的子元素上。添加到`g`元素的属性会被其所有的子元素继承。此外，`g`元素也可以用来定义复杂的对象，之后可以通过[`use`](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Element/use)元素来引用它们。

```html
<svg xmlns="http://www.w3.org/2000/svg"
     width="800" height="600" viewbox="0 0 800 600">
    <g stroke="green" fill="none"
       transform="translate(0,50)">
    	<rect x="100" y="50"
              width="100" height="50">
        </rect>
        <rect x="140" y="100"
              width="20" height="120">
        </rect>
    </g>
</svg>
```

可以看到`g`标签将两个矩形包裹在一起，`g`标签上的属性会被被包裹的元素继承。

我们还可以使用`symbol`标签来组合元素，`symbol`元素永远都不会显示，这一点和`g`不同。`symbol`可以指定独立地`viewbox`和`preserveAspectRatio`属性，以方便外部封装，通过给`use`标签指定`width`和`height`就可以让`symbol`适配视窗大小。

## transform变换

可以使用`transform`属性对图形进行变换，支持的类型有：

- **Translate**  `translate(x y)` 变换函数通过 `x` 向量和 `y` 向量移动元素，如果 `y` 向量没有被提供，那么默认为 `0`  。
- **Scale**  `scale(x y)` 变换函数通过 `x` 和 `y`指定一个 **等比例放大缩小** 操作。如果 `y` 没有被提供，那么假定为等同于 `x` 。
- **Rotate**  `rotate(x y)` 变换方法通过一个给定角度对一个指定的点进行旋转变换。如果x和y没有提供，那么默认为当前元素坐标系原点。否则，就以`(x,y)`为原点进行旋转。 
- **SkewX**  `skewX(a)` 变换函数指定了沿 `x` 轴倾斜 `a°` 的倾斜变换。 
- **SkewY**  `skewY(a)` 变换函数指定了沿 `y` 轴倾斜 `a°` 的倾斜变换。
- **Matrix**  `matrix(a b c d e f)` 函数以六个值的变换矩阵形式指定一个 `transform`。 

更多具体信息可见[transform - SVG|MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/transform)。



## RGB和HSL

### RGB

- 红色、绿色、蓝色三个分量
- 表示方式：`rgb(r, g, b)`或`#rrggbb`
- 每个分量取值范围： `[0, 255]`
- 优势： 显示器容易解析
- 劣势： 不符合人类描述颜色的习惯

### HSL

- 三个分量分别表示颜色、饱和度和亮度
- 表示方式：`hsl(h, s%, l%)`
- 取值范围：
  - `h: [0, 359]`
  - `s, l: [0, 100]`
- 优势： 符合人类描述颜色的习惯

### 透明度

- `rgba(r, g, b, a)`和`hsla(h, s%, l%, a)表示带透明度的颜色`
- `opacity`属性表示元素的透明度
- `a`和`opacity`的取值范围`[0, 1]`

### 在SVG中使用颜色

```html
<rect fill="rgb(255,0,0)" opacity=".5"/>
<rect stroke="hsla(0,50%,60%,.5)"/>
```

## 渐变

>  渐变是一种从一种颜色到另一种颜色的平滑过渡。另外，可以把多个颜色的过渡应用到同一个元素上。 

在 `SVG` 中，有两种主要的渐变类型：

- 线性渐变
- 径向渐变

### 线性渐变

 <linearGradient> 可用来定义 `SVG `的线性渐变。 

<linearGradient> 标签必须嵌套在 <defs> 的内部。<defs> 标签是 `definitions `的缩写，它可对诸如渐变之类的特殊元素进行定义。

`x1`、`y1`、`x2`、`y2`定义了渐变开始和结束的位置，线性渐变可被定义为水平、垂直或角形的渐变：

- 当` y1 `和 `y2 `相等，而 `x1 `和 `x2 `不同时，可创建水平渐变
- 当 `x1` 和 `x2 `相等，而` y1 `和 `y2 `不同时，可创建垂直渐变
- 当 `x1 `和 `x2 `不同，且` y1` 和 `y2 `不同时，可创建角形渐变

 一个渐变上的颜色坡度，是用`stop`标签定义的， `offset`属性用来定义该色块的开始和结束位置。 形如：

```html
<svg xmlns="http://www.w3.org/2000/svg"
     width="200" height="150">
    <defs>
        <linearGradient id="grad1" gradientUnits="objectBoundingBox" x1="0"
                        y1="0" x2="1" y2="1">
            <stop offset="0" stop-color="#1497FC" />
            <stop offset="0.5" stop-color="#A469BE" />
            <stop offset="1" stop-color="#FF8C00" />
        </linearGradient>
    </defs>
    <rect x="0" y="0" fill="url(#grad1)" width="200" height="150"/>
</svg>

```

在图形中填充使用`url(#id)"`进行使用。

在渐变的属性中有一个`gradientUnits`，默认为`objectBoundingBox`，以包围渐变的盒子来描述两个端点的坐标，`x`的`0`为最左端，`1`为最右端，`y`的`0`为最上端，`1`为最下端。另一个取值为`userSpaceOnUse`，设置后会使用标准的坐标系来描述坐标。

### 径向性渐变

<radialGradient> 用来定义径向渐变。

<radialGradient> 标签必须嵌套在 <defs> 中。<defs> 标签是 `definitions` 的缩写，它允许对诸如渐变等特殊元素进行定义。

一个例子：

```html
<svg xmlns="http://www.w3.org/2000/svg" width="200" height="150">
    <defs>
        <radialGradient id="grad2" cx="0.5"
                        cy="0.5" r="0.5" fx="0.6" fy="0.3">
            <stop offset="0" stop-color="rgb(20,151,252)" />
            <stop offset="0.5" stop-color="rgb(164,105,190)" />
            <stop offset="1" stop-color="rgb(255,140,0)" />
        </radialGradient>
    </defs>
    <rect x="0" y="0" fill="url(#grad2)" width="200" height="150"/>
</svg>
```

`cx`、`cy`表示渐变开始的坐标，`r`表示渐变半径，`fx`、`fy`定义焦点坐标（没有则假定为中心点）。

## 笔刷

笔刷主要是绘制一个纹理，然后可以进行重复的填充，`SVg`中使用<pattern>标签来定义一个笔刷（定义在<defs>中），当一个图形用笔刷来填充时，就会不断将笔刷的图形填充到目标图形中。

```html
<svg xmlns="http://www.w3.org/2000/svg" width="400" height="300">
    <defs>
        <pattern id="p1" x="0"
                        y="0" width="0.2" height="0.2">
            <circle cx="10" cy="10" r="5" fill="red"></circle>
            <polygon points="30 10 60 50 0 50" fill="green"></polygon>
        </pattern>
    </defs>
    <rect x="0" y="0" fill="url(#p1)" width="400" height="300" stroke="blue"/>
</svg>
```

<pattern>的宽和高都为`0.2`，这时不管我们如何更改<rect>的大小，笔刷图案永远都只会是`25`个，只是间隔会改变而已。当我们把属性`patternUnits`跟改为`userSpaceOnUse`，笔刷的宽和高就会使用标准坐标系进行定义：

```html
<svg xmlns="http://www.w3.org/2000/svg" width="400" height="300">
    <defs>
        <pattern id="p1" x="0"
                        y="0" width="60" height="50" patternUnits="userSpaceOnUse">
            <circle cx="10" cy="10" r="5" fill="red"></circle>
            <polygon points="30 10 60 50 0 50" fill="green"></polygon>
        </pattern>
    </defs>
    <rect x="0" y="0" fill="url(#p1)" width="400" height="300" stroke="blue"/>
</svg>
```

这时就会按照每个笔刷的大小不限个数地填充整个图形。

同样还有另一个属性`patternContentUnits`来规定<pattern>中的图形宽高等属性的内容单位定义，当为`objectBoundingBox`时，<pattern>中的图形宽高等属性按填充图形的百分比来定义，取值范围`[0-1]`，而`userSpaceOnUse`则按照标准坐标系（默认）。

可以理一下`patternUnits`和`patternContentUnits`的关系，前者针对笔刷本身，后者针对笔刷内的内容。

## path

 <path>是用来定义形状的通用元素 ，使用<path>可以绘制任意的几何图形，形如：

```html
<path d="M0,0L10,20C30-10,40,20,100,100" stroke="red">
```

属性`d`定义了一条路径，它是一个字符串，包含了一系列路径的描述，路径由一系列命令序列组成，如`L10,20`，`L`是命令，`10,20`是参数。在命令和参数之间可以随意用空格或者逗号隔开，有一种情况例外，就是下一个数值是负数，负数可以表示上一个参数的结束，所以可以不用空格或者逗号。

### 命令

| 命令                      | 含义                                       |
| ------------------------- | ------------------------------------------ |
| M/m (x,y)+                | 移动当前位置                               |
| L/l (x,y)+                | 从当前位置绘制线段到指定位置               |
| H/h (x)+                  | 从当前位置绘制水平线到达指定的X坐标        |
| V/v (y)+                  | 从当前位置绘制竖直线到达指定的y坐标        |
| Z/z                       | 闭合当前路径                               |
| C/c (x1,y1,x2,y2,x,y)+    | 从当前位置绘制三次贝塞尔曲线到指定位置     |
| S/s (x2,y2,x,y)+          | 从当前位置光滑绘制三次贝塞尔曲线到指定位置 |
| Q/q (x1,y1,x,y)+          | 从当前位置绘制二次贝塞尔曲线到指定位置     |
| T/t (x,y)+                | 从当前位置光滑绘制二次贝塞尔曲线到指定位置 |
| A/a (rx,ry,xr,laf,sf,x,y) | 从当前位置绘制弧线到指定位置               |

**命令的基本规律**

- 区分大小写：大写表示坐标参数为绝对位置，小写则为相对位置（相对上一次画笔停留的位置）
- 最后的参数表示最终要到达的位置
- 上一个命令结束的位置就是下一个命令开始的位置
- 命令可以重复参数表示重复执行同一条命令

#### 移动和直线命令

- `M（x,y)+` 移动画笔，后面如果有重复参数，会当做是L命令处理 ，在没有绘制时，画笔的初始位置为`（0,0）`，如果进行了绘制，绘制完成后的最后位置就是画笔的位置。
- `L （x,y)+ `就是绘制直线到指定位置。
- `H （x)+` 绘制水平线到指定的x位置，因为是绘制水平线，所以只需要x坐标即可。
- `V （y）+ `绘制竖直线到指定的y位置，因为是绘制垂直线，所以只需要y坐标即可。
- `m、l、h、v`使用相对位置绘制。

#### 弧线命令

- `A （rx,ry,xr,laf,sf,x,y）`绘制弧线。
- 参数详解
  - `rx` - `(radius-x)` 弧线所在椭圆的`x`半轴长，以坐标系画圆，`rx`为`x`轴的半径长
  - `ry` - `(radius-y)` 弧线所在椭圆的`y`半轴长，同上，为`y`轴半径长
  - `xr` - `(xAxis-rotaion)` 弧线所在的椭圆的长弧角度（可以理解为`x`坐标的旋转角度）
  - `laf` - `(large-arc-flag)`是否选择弧长较长的那一段弧，`0`小弧，`1`大弧
  - `sf` - `(sweep-flag)` 是否选择逆时针方向的那一段弧，`0`逆时针，`1`顺时针
  - `x,y` -  弧的终点位置

### 贝塞尔曲线

#### 二次贝塞尔曲线

- `Q x1 y1 x y`绘制二次贝塞尔曲线。
- 参数详解
  - `x1`  控制点的`x`坐标
  - `y1` 控制点的`y`坐标
  - `x` 结束点的`x`坐标
  - `y` 结束点的`y`坐标

- 起始点就是画笔最后的落点。

#### 三次贝塞尔曲线

- `C x1 y1 x y`绘制三次贝塞尔曲线。
- 参数详解
  - `x1`  控制点一的`x`坐标
  - `y1` 控制点一的`y`坐标
  - `x2` 控制点二的`x`坐标
  - `y2` 控制点二的`y`坐标
  - `x` 结束点的`x`坐标
  - `y` 结束点的`y`坐标

- 起始点就是画笔最后的落点。

#### 光滑曲线

- `T`： `Q`的光滑版本。
  - 控制点可以省略，它是上一段曲线的控制点关于当前曲线起始点的镜像位置
- `S`： `C`的简化版本。
  - 第一个控制点可以省略，它是上一段曲线的第二个控制点关于当前曲线起始点的镜像位置

## 文本

### text和tspan标签

 `text`标签定义了一个由文字组成的图形。注意：我们可以将渐变、图案、剪切路径、遮罩或者滤镜应用到`text`上。 

- `x`和`y`属性 -  坐标定位
- `dx`和`dy`属性 - 字形偏移
- `style`属性 - 设置样式

当`dx`和`dy`是一组数时，比如`dx="20 40 80 100"`，那么文字不仅会偏移，还会在每个文字之间增加对应的间距，比如文字是`ABCD`，那么`A`开头会有`20`的间距，`A`与`B`之间会有`40`的间距，以此类推。



 在 `text`标签中，利用内含的`tspan`标签，可以调整文本和字体的属性以及当前文本的位置、绝对或相对坐标值。 可以理解为使用`tspan`包裹的文字可以进行单独的设置，类似于`html`中`p`标签之内包裹`span`的操作。

### 路径文本

 除了笔直地绘制一行文字以外， `SVG` 也可以根据`path`标签的形状来放置文字。 只要在`textPath`标签内部放置文本，并通过其`xlink:href`属性值引用`path`，我们就可以让文字块呈现在`path`标签给定的路径上了。 

```html
<path id="path1" d="M 100 200 Q 200 100 300 200 T 500 200" stroke = "rgb(0,255,0)" fill="none"/>

<text style="font-size: 24px">
	<textPath xlink:href="#path1">
    	这个文字先上去，又下来了。 Upside down in english!
    </textPath>
</text>
```

文字长度超出路径长度则不进行渲染。

### 超链接

 使用 `SVG `的锚元素 <a>定义一个超链接。 

- 可以添加到任意图形上
- `xlink:href`指定链接地址
- `xlink:title`指定链接提示
- `target`指定打开目标方式，如`__blank`

```html
<svg xmlns="http://www.w3.org/2000/svg" width="800" height="600">
    <a 
       xlink:href="http://baike.baidu.com/search/word?word=正方形"
       xlink:title="正方形"
       target="_blank">
        <rect
              x="100"
              y="100"
              width="100"
              height="100"
              fill="rgba(255, 0, 0, .5)"
              stroke="rgb(200, 0, 0)"
              stroke-width="2">
        </rect>
    </a>
</svg>
```

## 图形引用、裁切和蒙版

### defs标签创建图形引用

 <defs>标签允许我们定义以后需要重复使用的图形元素。 建议把所有需要再次使用的引用元素定义在`defs`元素里面。这样做可以增加`SVG`内容的易读性和可访问性。 在`defs`元素中定义的图形元素不会直接呈现。 你可以在你的视口的任意地方利用 `use`元素呈现这些元素。 

```html
<svg width="80px" height="30px" viewBox="0 0 80 30"
     xmlns="http://www.w3.org/2000/svg">

  <defs>
    <linearGradient id="Gradient01">
      <stop offset="20%" stop-color="#39F" />
      <stop offset="90%" stop-color="#F3F" />
    </linearGradient>
  </defs>

  <rect x="10" y="10" width="60" height="10" 
        fill="url(#Gradient01)"  />
</svg>
```



### use标签使用图形引用

 `use`元素在`SVG`文档内取得目标节点，并在别的地方复制它们。它的效果等同于这些节点被深克隆到一个不可见的`DOM`中，然后将其粘贴到`use`元素的位置，很像`HTML5`中的克隆[模板元素](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template)。因为克隆的节点是不可见的，所以当使用[CSS](https://developer.mozilla.org/en/CSS)样式化一个`use`元素以及它的隐藏的后代元素的时候，必须小心注意。隐藏的、克隆的`DOM`不能保证继承`CSS`属性，除非你明文设置使用[CSS继承](https://developer.mozilla.org/en/CSS/inheritance)。 

```html
<svg width="100%" height="100%" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
  <style>
    .classA { fill:red }
  </style> 
  <defs>
    <g id="Port">
      <circle style="fill:inherit" r="10"/>
    </g>
  </defs>
 
  <text y="15">black</text>
  <use x="50" y="10" xlink:href="#Port" />
  <text y="35">red</text>
  <use x="50" y="30" xlink:href="#Port" class="classA"/>
  <text y="55">blue</text>
  <use x="50" y="50" xlink:href="#Port" style="fill:blue"/>
 </svg>
```

### clipPath标签裁剪图形

`clipPath`标签定义一条剪切路径，可作为其他元素的 `clip-path` 属性的值。

剪切路径限制了图形的可见范围。从概念上来说，如果图形超出了当前剪切路径所包围的区域，那么超出部分将不会绘制。

```html
<svg viewBox="0 0 100 100">
  <clipPath id="myClip">
    <!--
      这个圆圈外的所有东西都会被裁剪掉，因此不可见。
    -->
    <circle cx="40" cy="35" r="35" />
  </clipPath>
 
  <!-- 作为引用元素的黑色心形 -->
  <path id="heart" d="M10,30 A20,20,0,0,1,50,30 A20,20,0,0,1,90,30 Q90,60,50,90 Q10,60,10,30 Z" />
 
  <!--
    和上述黑色心形形状相同的红色心形，剪切路径是上面定义的圆；
    红色心形只有在圆内的部分可见。
  -->
  <use clip-path="url(#myClip)" xlink:href="#heart" fill="red" />
</svg>
```

```css
/* 如果浏览器支持几何属性 r，可以加一点 css */

@keyframes openYourHeart {from {r: 0} to {r: 60px}}

#myClip circle {
  animation: openYourHeart 15s infinite;
}
```



### mask标签创建蒙版

> 蒙版中黑色代表不可见（opacity: 0），白色代表可见（opacity: 100%）。从0%到100%是一个线性的变化，所以黑白中间的灰色会是半透明，不同灰度代表不同程度的半透明，越趋近白色可见度越高。在蒙版中的黑白渐变，应用到彩色图层上就会产生透明度的渐变 。

 使用黑白蒙版来制作一轮弯月 ：

首先制作黑白蒙版:

```html
<svg height="70" style="background:gray" version="1.1" xmlns="http://www.w3.org/2000/svg" >
  <circle cx="25" cy="25" r="25" fill="white"/>
  <circle cx="40" cy="15" r="25" fill="black"/>
</svg>
```

接下来，将上面的两个circle元素制作为蒙版，并应用在一个圆形上，这样就制作出了一轮弯月：

```html
<svg height="70" style="background:gray" version="1.1" xmlns="http://www.w3.org/2000/svg" >
  <mask id="moon-mask">
    <circle cx="25" cy="25" r="25" fill="white"/>
    <circle cx="40" cy="15" r="25" fill="black"/>    
  </mask>
  <circle cx="25" cy="25" r="25" fill="yellow" mask="url(#moon-mask)"/>
</svg>
```

## 动画

### 定位动画目标

- Internal Resource Identifier 定位

```html
<animate xlink:href="url(#rect1)"></animate>
```

- 被包含在目标元素里

```html
<rect c="0" ...>
	<animate></animate>
</rect>
```

### 基本动画

- 设置要进行动画的属性以及变化的范围、时间长度

```html
<!--attributeType: 指定attributeName的值是什么属性名
，取值可以为CSS、XML、auto，
也就是说可以指定为css属性，也可以指定为xml属性，
指定为auto时，用户代理先搜索CSS属性列表以找出一个匹配的属性名，如果找不到，再为这个元素搜索默认XML命名空间。
-->

<!--fill属性作用在animate上时，默认值为remove，表示在动画的激活结束后，动画不再对目标元素有影响，回到开始前的状态（除非动画重新开始）。
freeze表示在动画激活持续时间结束后，文档持续时间的剩余时间里（或者直到动画重新开始）动画效果会“冻结”着，保持最后的状态。
-->

<!--begin指定动画何时开始，默认是0，则加载后立即开始。它可以指定以下值：

合法的时钟值：
02:30:03 = 2小时30分钟又3秒
50:00:10.25 = 50小时10秒又250毫秒
部分时钟值：
02:33 = 2分钟又33秒
00:10.5 = 10.5 秒 = 10秒又500毫秒
Timecount值：
3.2h = 3.2小时 = 3小时12分钟
45min = 45分钟
30s = 30秒
5ms = 5毫秒
12.467 = 12秒又467毫秒

<syncbase-value>
描述一个syncbase以及一个可选的来自于syncbase的时偏移。元素的动画开始时间被定义为相对于另一个动画的开头或者激活结束。一个ID及其后面跟着的.begin或.end构成了一个syncbase，ID引用到另一个动画元素（id不要用-号分隔，否则可能会动画失败），.begin或.end用来确定到底是与引用的动画元素的动画开头同步、 还是与引用的动画元素的动画激活结束同步。

<event-value>
描述了一个事件以及一个可选的时偏移，用来确定动画开始的时点。触发指定事件的时点，被定义为动画开始的时点。一个元素ID及其后面跟着的一个点及其后面跟着事件名构成了一个合法的event-value值。事件名必须是元素支持的事件名。全部合法的事件（不一定是所有元素都支持的事件）包括这些：focusin、 focusout、 activate、 click、 mousedown、 mouseup、 mouseover、 mousemove、 mouseout、 DOMSubtreeModified、 DOMNodeInserted、 DOMNodeRemoved、 DOMNodeRemovedFromDocument、 DOMNodeInsertedIntoDocument、 DOMAttrModified、 DOMCharacterDataModified、 SVGLoad、 SVGUnload、 SVGAbort、 SVGError、 SVGResize、 SVGScroll、 SVGZoom、 beginEvent、 endEvent和repeatEvent。

<repeat-value>
描述了一个符合条件重复事件。repeat事件发生了指定次数的时间点，被定义为元素动画的开始时间点。

<accessKey-value>
描述了一个用来触发动画的访问键。当用户按下指定的键时，元素动画就开始了。

<wallclock-sync-value>
描述了一个动画开始时间，是真实世界钟的时点。时点的句法遵守ISO8601定义的句法。

begin是可以指定多个值的，用;号分隔，比如"begin:0;goleft.end"，表示立即开始，同时在goleft这个id的元素动画结束后开始。"begin:0;goleft.end + 2s"表示立即开始，并且在goleft这个元素动画结束2s后开始。
-->

<!--repeatcount值必须大于0，indefinite表示无限循环-->

<!--动画是可以叠加的，也就是说一个元素可以使用多个animate标签, 添加属性`additive: "sum"`，additive默认值为`replace`表示替换，设置为sum表示可叠加。-->

<animate xlink:href="url(#rect1)"       
         attributeType="XML"
         attributeName="x"
         begin="0"
         from="10"
         to="110"
         dur="3s"
         fill="freeze"
         repeatCount="indefinite"></animate>
```

### 变换动画

- 设置要进行动画的属性以及变化范围、时间长度

```html
<!--type: translate | scale | rotate | skewX | skewY -->

<svg width="120" height="120"  viewBox="0 0 120 120"
     xmlns="http://www.w3.org/2000/svg" version="1.1"
     xmlns:xlink="http://www.w3.org/1999/xlink" >

    <polygon points="60,30 90,90 30,90">
        <animateTransform attributeName="transform"
                          attributeType="XML"
                          type="rotate"
                          from="0 60 70"
                          to="360 60 70"
                          dur="10s"
                          repeatCount="indefinite"/>
    </polygon>
</svg>
```

### 轨迹移动

 <animateMotion> 元素定义了一个元素如何沿着运动路径进行移动。 

```css
html,body,svg { height:100%; margin: 0; padding: 0; display:block; }
```

```html
<svg viewBox="0 0 200 100" xmlns="http://www.w3.org/2000/svg">
  <path fill="none" stroke="lightgrey"
    d="M20,50 C20,-50 180,150 180,50 C180-50 20,150 20,50 z" />

  <circle r="5" fill="red">
    <animateMotion dur="10s" repeatCount="indefinite"
      path="M20,50 C20,-50 180,150 180,50 C180-50 20,150 20,50 z" />
  </circle>
</svg>
```

 **注意：**为了复用一个已经定义的路径，就有必要使用一个`mpath` 元素嵌入到 `animateMotion`中，而不是使用 `path`。 

```html
<svg width="100%" height="100%"  viewBox="0 0 500 300"
     xmlns="http://www.w3.org/2000/svg"
     xmlns:xlink="http://www.w3.org/1999/xlink" >

  <rect x="1" y="1" width="498" height="298"
        fill="none" stroke="blue" stroke-width="2" />

  <!-- Draw the outline of the motion path in blue, along
          with three small circles at the start, middle and end. -->
  <path id="path1" d="M100,250 C 100,50 400,50 400,250"
        fill="none" stroke="blue" stroke-width="7.06"  />
  <circle cx="100" cy="250" r="17.64" fill="blue"  />
  <circle cx="250" cy="100" r="17.64" fill="blue"  />
  <circle cx="400" cy="250" r="17.64" fill="blue"  />

  <!-- Here is a triangle which will be moved about the motion path.
       It is defined with an upright orientation with the base of
       the triangle centered horizontally just above the origin. -->
  <path d="M-25,-12.5 L25,-12.5 L 0,-87.5 z"
        fill="yellow" stroke="red" stroke-width="7.06"  >
    <!-- Define the motion path animation -->
    <animateMotion dur="6s" repeatCount="indefinite" rotate="auto" >
       <mpath xlink:href="#path1"/>
    </animateMotion>
  </path>
</svg>
```