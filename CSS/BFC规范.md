# BFC

## BFC是什么

Block fomatting context = block-level box + Formatting Context

### Box

　　Box即盒子模型；

### block-level box即块级元素

display属性为block, list-item, table的元素，会生成block-level box；并且参与 block fomatting context；

### Formatting context

　　Formatting context是W3C CSS2.1规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系、相互作用。最常见的 Formatting context 有 Block fomatting context (简称BFC)和 Inline formatting context(简称IFC)。

　　CSS2.1 中只有BFC和IFC, CSS3中还增加了G（grid）FC和F(flex)FC。

## BFC定义

BFC(Block formatting context)直译为"块级格式化上下文"。它是一个独立的渲染区域，只有Block-level box参与， 它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干，可以简单的理解为创建了一个局部作用域。

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

````HTML
<!DOCTYPE html>
<html lang="en">
<body>
    <div class="main">
        <div class="content"></div>
        <div class="content"></div>
        <div class="content"></div>
        <div class="floatLeft"></div>
    </div>
</body>
<style>
    .main {
        /* 触发BFC */
        overflow: hidden;
        width: 120px;
        background: blue;
        border: 1px solid blue;
    }
    .content {
        background: red;
        width: 20px;
        height: 20px;
        margin: 20px;
    }
    .floatLeft {
        float: left;
        margin-left: 20px;
        width: 20px;
        height: 20px;
        background: green;
    }
</style>
</html>
````

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c0b79942e2d64b4dbc6547a201964b2a~tplv-k3u1fbpfcp-zoom-1.image)

- BFC的区域不会与float的元素区域重叠

不触发BFC,当floatLeft浮动的时候会与conetent重叠

````html
<!DOCTYPE html>
<html lang="en">
<body>
    <div class="main">
        <div class="floatLeft"></div>
        <div class="content"></div>
    </div>
</body>
<style>
    .content {
        background: red;
        width: 100px;
        height: 100px;
    }
    .floatLeft {
        float: left;
        width: 40px;
        height: 40px;
        background: green;
    }
</style>
</html>
````

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09b6d68cfd3146ffb69992ad9320a678~tplv-k3u1fbpfcp-zoom-1.image)

如果触发BFC，content需要保持独立的作用域，因此float的元素不会与触发BFC的元素重叠

````html
<!DOCTYPE html>
<html lang="en">
<body>
    <div class="main">
        <div class="floatLeft"></div>
        <div class="content"></div>
    </div>
</body>
<style>
    .content {
        /* 触发BFC */
        overflow: hidden;
        background: red;
        width: 100px;
        height: 100px;
    }
    .floatLeft {
        float: left;
        width: 40px;
        height: 40px;
        background: green;
    }
</style>
</html>
````

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4600d017cba4535b0a2afbf27501e8a~tplv-k3u1fbpfcp-zoom-1.image)

- 计算BFC的高度时，浮动子元素也参与计算
- BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素，反之亦然

这两条可以结合一起看，普通的情况下，浮动的子元素不参与高度计算，导致父元素高度坍塌

````html
<!DOCTYPE html>
<html lang="en">
<body>
    <div class="main">
        <div class="content"></div>
    </div>
</body>
<style>
    .main {
        background: blue;
        border: 1px solid blue;
    }
    .content {
        float: left;
        background: red;
        width: 100px;
        height: 20px;
    }
</style>
</html>
````

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09b066a9852f486f874da7b5213948a9~tplv-k3u1fbpfcp-zoom-1.image)

当触发BFC时，由于`BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素`，为了不让子元影响到外面元素，所以父元素会把浮动元素包裹起来，即`计算BFC的高度时，浮动子元素也参与计算`

````html
<!DOCTYPE html>
<html lang="en">
<body>
    <div class="main">
        <div class="content"></div>
    </div>
</body>
<style>
    .main {
        /* 触发BFC */
        overflow: hidden;
        background: blue;
        border: 1px solid blue;
    }
    .content {
        float: left;
        background: red;
        width: 100px;
        height: 20px;
    }
</style>
</html>
````

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9a47459ee5e147139849b4fa6c05a625~tplv-k3u1fbpfcp-zoom-1.image)

## 外边距折叠（margin collapse）

先来看看外边距折叠的表现

````html
<!DOCTYPE html>
<html lang="en">
<body>
    <p></p>
    <p></p>
    <p></p>
</body>
<style>
    p {
        background: red;
        height: 20px;
        margin: 20px 0;
    }
</style>
</html>
````

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/89ae0f4a0c8b41ff9a4e2998e375d064~tplv-k3u1fbpfcp-zoom-1.image)

为什么设置margin之后的间距不是40px，而是20px呢？

原因是因为在css出现之前，并没有特别的手段去自定义样式，排版全靠标签自带的样式，p标签之间天然拥有相同的间隔。在css出现之后，为了和老版的p标签保持一致，便规定了一个特定的规则--margin collapse

那什么情况下会发生外边距折叠呢？

- 都是普通流中的元素且属于同一个 BFC
- 没有被 padding、border、clear 或非空内容隔开
- 两个或两个以上垂直方向的「相邻元素」

注意这里的「相邻元素」可能是兄弟节点也可能是父子节点，比如：一个元素的 margin-top 和它的第一个普通流子元素的 margin-top；一个元素的 margin-bottom 和它下一个普通流兄弟的 margin-top；一个高度为 auto 元素的 margin-bottom 和它的最后一个子元素的 margin-bottom

如果要避免margin collapse，只需要破上面的规则即可

我们来看看使用BFC来破坏margin collapse的例子

````html
<!DOCTYPE html>
<html lang="en">
<body>
    <div class="BFC">
        <p></p>
    </div>
    <div class="BFC">
        <p></p>
    </div>
    <div class="BFC">
        <p></p>
    </div>
</body>
<style>
.BFC {
    /* 触发BFC */
    overflow: hidden;
}
p {
    background: red;
    height: 20px;
    margin: 20px 0;
}
</style>
</html>
````

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/36613945c89147d2af42514409ac3ca7~tplv-k3u1fbpfcp-zoom-1.image)
