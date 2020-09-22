## BFC是什么？
Block fomatting context = block-level box + Formatting Context

### Box:
　　Box即盒子模型；

### block-level box即块级元素
display属性为block, list-item, table的元素，会生成block-level box；并且参与 block fomatting context；

### inline-level box即行内元素
display 属性为 inline, inline-block, inline-table的元素，会生成inline-level box。并且参与 inline formatting context；

### Formatting context
　　Formatting context是W3C CSS2.1规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系、相互作用。最常见的 Formatting context 有 Block fomatting context (简称BFC)和 Inline formatting context(简称IFC)。

　　CSS2.1 中只有BFC和IFC, CSS3中还增加了G（grid）FC和F(flex)FC。

## BFC定义

BFC(Block formatting context)直译为"块级格式化上下文"。它是一个独立的渲染区域，只有Block-level box参与， 它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干。

## BFC如何生成

- 根元素
- float的值不为none
- overflow的值不为visible
- display的值为inline-block、table-cell、table-caption
- display：table也认为可以生成BFC，其实这里的主要原因在于Table会默认生成一个匿名的table-cell，正是这个匿名的table-cell生成了BFC
- position的值为absolute或fixed

### BFC的约束规则

- 内部的Box会在垂直方向上一个接一个的放置
- 垂直方向上的距离由margin决定。（完整的说法是：属于同一个BFC的两个相邻Box的margin会发生重叠（塌陷），与方向无关。）
- 每个元素的左外边距与包含块的左边界相接触（从左向右），即使浮动元素也是如此。（这说明BFC中子元素不会超出他的包含块，而position为absolute的元素可以超出他的包含块边界）
- BFC的区域不会与float的元素区域重叠
- 计算BFC的高度时，浮动子元素也参与计算
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素，反之亦然
看到以上的几条约束，想想我们学习css时的几条规则

- Block元素会扩展到与父元素同宽，所以block元素会垂直排列
- 垂直方向上的两个相邻DIV的margin会重叠，而水平方向不会(此规则并不完全正确)
- 浮动元素会尽量接近往左上方（或右上方）
- 为父元素设置overflow：hidden或浮动父元素，则会包含浮动元素