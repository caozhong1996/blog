# commonmark 推荐解析算法

## 概述

解析过程有两个阶段：

1. 将输入文本断行，顺序解析每一行并生成块级节点（段落，块引用，列表项等）。文本作为块级节点的内容，暂时不进行解析。链接引用定义在这个阶段也会被解析构造放到一个 Map 中。

    > 块级解析算法是基于行的，每次读取一行进行处理，断行的依据是 \r\n 和 \n，因为规范中定义了行结束符就是这两者。Lute 是在词法分析阶段做了预处理，将 \r\n 替换为 \n，后续统一使用 \n 进行断行。

2. 第二阶段，将段落与标题的原生文本解析为行内元素（字符串、内联代码、 链接，强调）等。链接使用第一阶段的 Map 进行解析。

在处理过程中，文档视为一个块树。树的根是 `document` 块。`document` 可包含任意其它块作为它的子元素。反过来，这些子元素可能还有其他的块作为子元素。块的最后一个子元素通常是打开的，后面输入行能改变它的内容。例如，下面是一个文档树，**箭头标记**的块是打开的：

```tree
-> document
  -> block_quote
       paragraph
         "Lorem ipsum dolor\nsit amet."
    -> list (type=bullet tight=true bullet_char=-)
         list_item
           paragraph
             "Qui *quodsi iracundia*"
      -> list_item
        -> paragraph
             "aliquando id"
```

## 块结构

每一行在解析时都会影响文档树。根据行的内容分析行 将以下面一种或数种方式 改变文档：

1. 一或多个打开的块被关闭。
2. 一或多个新块被创建为最后打开的块的子元素。
3. 文本添加给最后打开的块。

一旦行合并到树中便可以抛弃，因此输入能以流的方式读取。

每一行都遵循如下步骤：

1. 首先我们循环访问打开的块节点，从根节点（root document）开始，下降到最后一个子块，直到最后一个打开的块。每个一个块要打开都要满足基本条件。例如，块引号（block quote）需要一个 `>` 字符。段落需要一个非空行。在这个阶段，我们可以匹配所有或者只是一些打开的块。但是我们还不能关闭不匹配的块，因为我们可能还有一个惰性的[延续行](https://spec.commonmark.org/0.30/#lazy-continuation-line)。

2. 接下来，在使用现有块的延续标记之后，我们查找新的块启动标识（例如 块引号的 `>`）。如果我们遇到一个新块的标识，在创建新块作为最后匹配容器块的子元素之前，需要关闭步骤1中任何不匹配的块

3. 最后，我们查看行的其余部分（在 >，列表标记和缩进等块标记用完之后）。这是可以合并到最后一个打开的块（段落，代码块，标题或原始HTML）的文本。

当我们看到一行带有下划线的文本标题时，就会形成下划线。

在关闭段落时检测引用链接定义; 对堆积的文本行进行解析，以查看它们是否以一个或多个引用链接定义开始。任何剩余部分会视为一个正常的段落。

其中过程，让我们来看看下面四行 Markdown 如何生成上面的树：

```markdown
> Lorem ipsum dolor
sit amet.
> - Qui *quodsi iracundia*
> - aliquando id
```

在最外层，我们的文档模型仅仅是

```tree
-> document
```

第一行，

```markdown
> Lorem ipsum dolor
```

生成一个 `block_quote` 块，作为打开的 `document` 块的子元素。 并且一个 `paragraph` 块作为 `block_quote` 的子元素。 于是文本加到最后打开的块，`block_quote`：

```tree
-> document
  -> block_quote
    -> paragraph
         "Lorem ipsum dolor"
```

下一行，

```markdown
sit amet.
```

是打开的 `paragraph` 的惰性延续，所以添加到段落中：

```tree
-> document
  -> block_quote
    -> paragraph
         "Lorem ipsum dolor\nsit amet."
```

第三行，

```markdown
> - Qui *quodsi iracundia*
```

导致 `paragraph` 块关闭。一个新的 `list` 块打开为 `list` 的子元素， 一个 `paragraph` 为 `list_item` 的子元素。于是文本添加到这个新的 `paragraph`：

```tree
-> document
  -> block_quote
       paragraph
         "Lorem ipsum dolor\nsit amet."
    -> list (type=bullet tight=true bullet_char=-)
      -> list_item
        -> paragraph
             "Qui *quodsi iracundia*"
```

第四行，

```markdown
> - aliquando id
```

导致 `list_item` 及它的子元素 `paragraph` 关闭。一个新的 `list_item` 打开为 `list` 的子元素。一个 `paragraph` 添加为这个 `list_item` 的子元素，以包含文本。于是我们得到最终的树：

```tree
-> document
  -> block_quote
       paragraph
         "Lorem ipsum dolor\nsit amet."
    -> list (type=bullet tight=true bullet_char=-)
         list_item
           paragraph
             "Qui *quodsi iracundia*"
      -> list_item
        -> paragraph
             "aliquando id"
```

## 行内结构

输入一旦解析完成，所有的打开的块将关闭。

然后遍历树，访问每个节点，解析段落与标题的原生内容为行内元素。这时我们已能看到所有的链接引用定义，这样我们就可以解析参考链接了。

```tree
document
  block_quote
    paragraph
      str "Lorem ipsum dolor"
      softbreak
      str "sit amet."
    list (type=bullet tight=true bullet_char=-)
      list_item
        paragraph
          str "Qui "
          emph
            str "quodsi iracundia"
      list_item
        paragraph
          str "aliquando id"
```

注意第一个段落的[行结束符](/commonmark-spec/#line-ending)如何解析为软换行符。 第一个列表项中的星号变成了 `emph`。

根据渲染器，这个文档可渲染为 HTML 或其它格式。

## 一种解析嵌套强调和链接的算法

到目前为止，行内解析最棘手的部分是处理强调、加粗、链接、图片，这是使用以下算法完成的。

当我们解析行内

- 一串 `*` 或 `_` 字符， 或者
- 一个 `[` 或 `![`

则以这些符号作为文本内容生成一个文本节点，并在分隔符栈（delimiter stack）中压入一个指向该文本节点的元素。

分隔符堆栈是一个双向链表。每个元素包含一个指针指向一个文本节点，外加如下信息：

- 分隔符类型（[，![，*，_）
- 分隔符数量，比如强调是 1 个 *，加粗则为 2
- 分隔符是否处于“激活”状态（开始解析时都是激活状态）
- 分隔符是否是一个开始分隔符、结束分隔符或者两者都可能（这取决于分隔符前后的字符序列）

当我们解析时遇到 ]，则进入下面介绍的链接和图片处理过程。

当我们解析到输入结束时，则将 stack_bottom 置为 NULL 并进入下面介绍的强调处理过程。

### 链接和图片处理过程

从分隔符栈顶部开始回看寻找开始的 `[` 或者 `![` 分隔符元素。

- 如果没有找到，则返回一个文本节点`]`
- 如果找到了，但这个元素处于*非激活* 状态，则从栈中移除该元素，然后返回一个文本节点`]`
- 如果找到了，并且这个元素是激活的，则我们继续解析看是否能构成一个内联链接/图片、引用链接/图片、紧凑链接/图片或者快捷链接/图片
  - 如果不能，则从栈中移除这个开始分隔符，然后返回一个文本节点`]`
  - 如果能，则执行如下步骤
    - 生成一个链接或图片节点，其子元素为开始分隔符指向的文本节点之后的行级元素
    - 在这些行级元素上以开始分隔符`[` 作为`stack_bottom`  执行*强调处理过程*
    - 从栈中移除该开始分隔符
    - 如果是链接（不是图片），则设置所有位于该开始分隔符之前的`[` 为*非激活* 状态（防止链接嵌套链接）

#### 强调处理过程

参数 `stack_bottom` 设置了分隔符栈的栈底下限。如果其值为 NULL 则我们可以一直遍历到栈底。否则我们应该在访问到 `stack_bottom` 之前停止。

`current_position` 指向分隔符栈中高于 `stack_bottom` 的元素（当 `stack_bottom` 为 NULL 时指向第一个元素）。

使用 `openers_bottom` 来跟踪每种分隔符（按类型 `*`、`_` 和结束分隔符长度模 3）。初始化值为 `stack_bottom`。

然后我们重复以下步骤，直到用完了潜在的结束分隔符：

- 在分隔符栈中向前移动`current_position` 直到找到第一个潜在的结束分隔符`*` 或`_`。（这是离开始最近的结束分隔符 —— 也是按解析顺序的第一个）
- 现在我们向回查找（查找位置高于`stack_bottom` 以及相应的`openers_bottom` ）第一个匹配的开始分隔符（“匹配”的意思是和结束分隔符一样的分隔符）。
- 如果找到了：
  - 需要弄清楚是强调还是加粗：如果开始符和结束符的长度都 >=2，则是加粗，否则是普通强调
  - 在开始分隔符指向的文本节点后面插入一个 em 或者 strong 节点
  - 从分隔符栈中移除所有位于开始符和结束符之间的分隔符
  - 从开始和结束文本节点中移除 1 个（对于普通强调）或者 2 个（对于加粗）分隔符。如果它们为空了，则移除它们，并从分隔符栈中也进行移除。如果结束符节点被移除，则设置`current_position` 为栈中的下一个元素
- 如果没有找到：
  - 设置`openers_bottom` 指向`current_position` 前的元素。（此时我们知道该结束符没有对应的开始符，所以需要更新下限以便用于将来的搜索）
  - 如果`current_position` 指向的结束符不是一个潜在的开始符，则将它从分隔符栈中移除（因为它既不是开始符也不是结束符）
  - 将`current_position` 移动到栈中的下一个元素

处理完后，我们从分隔符栈中移除了位于 `stack_bottom` 之上的所有分隔符。
