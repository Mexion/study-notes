# 可替换元素

## 什么是可替换元素

我们都只到<img>标签是一个行内元素，通常而言，行内元素是不能设置宽高的，但是当我们尝试为<img>设置宽高时，却意外地发现它是能够成功设置宽高的，如下图：

![可替换元素-img](.\images\replaced-element.png)

这是为什么呢？

因为**<img>这个标签是一个可替换元素**，那么什么是可替换元素呢？

**可替换元素 就是 浏览器根据元素的标签和属性，来决定元素具体显示内容的元素。**

例如浏览器会根据<img>标签的src属性的值来读取图片信息并显示出来。

在 [CSS](https://developer.mozilla.org/zh-CN/docs/Web/CSS) 中，**可替换元素**（**replaced element**）的展现效果不是由 CSS 来控制的。这些元素是一种外部对象，它们外观的渲染，是独立于 CSS 的。简单来说，它们的内容不受当前文档的样式的影响。CSS 可以影响可替换元素的位置，但不会影响到可替换元素自身的内容。

典型的可替换元素有：

- [<iframe>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/iframe)
- [<video>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video)
- [embed](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/embed)
- [img](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img)

有些元素仅在特定情况下被作为可替换元素处理，例如：

- [option](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/option)
- [audio](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/audio)
- [canvas](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/canvas)
- [object](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/object)
- [applet](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/applet)

HTML 规范也说了 [input](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input) 元素可替换，因为 `"image"` 类型的 [input](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input) 元素就像[img](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img)一样被替换。但是其他形式的控制元素，包括其他类型的 [input](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input) 元素，被明确地列为非可替换元素（non-replaced elements）。该规范用术语小挂件（Widgets）来描述它们默认的限定平台的渲染行为。

用 CSS [`content`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/content) 属性插入的对象是匿名的可替换元素。它们并不存在于 HTML 标记中，因此是“匿名的”。

总之可替换元素虽然一般为行内元素，但是依然可以设置它们的宽高等属性。

