# 宽高比与尺寸控制教程

在绘图过程中，处理长宽比和其他精确控制图形的方法是一个非常常见的问题。

For example, many plots need square axes.
例如，许多图表需要正方形坐标轴。
如果你查阅过 `Axis` 的文档，可能会知道它有一个 `aspect` 属性，可以控制坐标轴框的长宽比。
这个长宽比与数据范围无关，只关乎坐标轴的相对视觉长度。
This aspect is not concerned with what the data limits are, it's just about the relative visual length of the axes.

让我们来看一个常见的例子，即带有颜色条的正方形坐标轴：

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
CairoMakie.activate!() # hide

set_theme!(backgroundcolor = :gray90)

f = Figure(size = (800, 500))
ax = Axis(f[1, 1], aspect = 1)
Colorbar(f[1, 2])
f
```

\end{examplefigure}

As you can see, the axis is square, but there's also a large gap between it and the colorbar.
Why is that?

我们可以通过在同一单元格中添加一个 `Box` 来可视化其原因：

\begin{examplefigure}{svg = true, name = "aspect_tutorial_example"}

```julia
Box(f[1, 1], color = (:red, 0.2), strokewidth = 0)
f
```

\end{examplefigure}

The red area of the box extends out into the whitespace left by the Axis.
This demonstrates what the `aspect` keyword is actually doing.
It reduces the size of the Axis, such that the chosen aspect ratio is achieved.
It doesn't tell the layout that the Axis lives in "please make this cell adhere to this aspect ratio".
As far as the layout is concerned, the Axis has an undefined size and its layout cell can therefore have any size that the layout deems correct, based on all other content of the layout and the figure size.

因此，使用 `aspect` 总会导致间隙，除非 Axis 所在的布局单元格碰巧恰好具有正确的长宽比。
这意味着只有当由此产生的空白不太重要的情况下才应使用 `aspect`。
This means `aspect` should only be used if the whitespace caused by it does not matter too much.

对于所有其他情况，有另一种方法。

我们希望强制布局保持轴单元格以特定的长宽比。
因此，我们需要操作布局本身，而不是轴。
Therefore, we have to manipulate the layout itself, not the axis.

默认情况下，每个 GridLayout 行和列的大小为`Auto()`。
这意味着如果有固定大小的内容，大小可能会取决于它，否则会扩展以填充可用空间。
如果我们想强制单元格具有长宽比，就需要将其相应的行或列大小设置为 `Aspect`。
This means that the size can depend on fixed-size content if there is any, otherwise it expands to fill the available space.
If we want to force a cell to have an aspect ratio, we need to set either its respective row or column size to `Aspect`.

让我们再次尝试上述示例，但这次我们将强制 Axis 所在的列相对于 Axis 所在的行（即第1行）具有1.0的长宽比。

\begin{examplefigure}{svg = true}

```julia
f = Figure(size = (800, 500))
ax = Axis(f[1, 1])
Colorbar(f[1, 2])
colsize!(f.layout, 1, Aspect(1, 1.0))
f
```

\end{examplefigure}

As you can see, this time the colorbar sticks close to the axis, there is no unnecessary whitespace between them.
如图所示，这次颜色条紧贴坐标轴，两者之间没有不必要的空白。
我们可以再次用红色 box 可视化 `Aspect` 的效果，显示布局单元格的范围：

\begin{examplefigure}{svg = true}

```julia
# hide
Box(f[1, 1], color = (:red, 0.2), strokewidth = 0)
f
```

\end{examplefigure}

So this time the layout cell itself is square, therefore the Axis that fills it is also square.
这次布局单元格本身是正方形，因此填充它的 Axis 也是正方形。
让我演示一下，我们可以再次玩同样的游戏，给Axis赋予不同于布局单元格正方形长宽比的 `aspect`，
这将再次导致不必要的空白：
This will again cause unnecessary whitespace:

\begin{examplefigure}{svg = true}

```julia
ax.aspect = 0.5
f
```

\end{examplefigure}

现在我们再次更改列的长宽比，以消除这个间隙：

\begin{examplefigure}{svg = true}

```julia
colsize!(f.layout, 1, Aspect(1, 0.5))
f
```

\end{examplefigure}

让我们回到先前的状态，即具有正方形坐标轴：

\begin{examplefigure}{svg = true}

```julia
# hide
f = Figure(size = (800, 500))
ax = Axis(f[1, 1])
Colorbar(f[1, 2])
colsize!(f.layout, 1, Aspect(1, 1.0))
f
```

\end{examplefigure}

现在你可能认为坐标轴与颜色条之间没有空白了，但实际上左右两侧有很多空白。
为什么布局不能为我们解决这个问题呢？
Why can the layout not fix this problem for us?

在 Makie 中，布局必须在我们设置的图形大小范围内运作。
如果内容不足，它不能简单地减小图形大小。
这是因为很多时候，图形是按照某些学术期刊的尺寸规则精确创建的，因此您所绘制的内容不允许影响图形大小。
It cannot just decrease the figure size if there's too little content.
This is because lots of times, figures are created to fit the sizing rules of some academic journal exactly, therefore the content you plot is not allowed to mess with the figure size.

在我们的示例中，我们对布局中对象的尺寸引入了约束，使得理论上可用的空间无法全部填满。
仔细想想，用一个正方形坐标轴和一个细长的颜色条填充这个矩形空间是不可能的。
我们需要一个更小的图形！
If you think about it, it's impossible to fill this Figure with a square axis and a thin colorbar while filling the rectangular space.
We need a smaller figure!

But how small should it be exactly?
It would be quite difficult to eyeball this, but thankfully there's a function for this exact purpose.
但它应该有多小才合适呢？
这可能会很难用肉眼观察，但幸运的是有一个专门用于此目的的函数。
通过调用`resize_to_layout!`，我们可以调整图形大小，使之符合布局中所有内容所需的大小。

让我们试试吧：

\begin{examplefigure}{svg = true}

```julia
resize_to_layout!(f)
f
```

\end{examplefigure}

As you can see, the whitespace at the sides has been trimmed.
(If the scaling looks smaller or bigger, that is just because of the display on this site, not the underlying figure size).

这种方法仅适用于所有“内容决定图形大小“的情况

For example, let's say we have a facet plot with 25 square axes which are all of size 150 by 150.
We can just make these axes with fixed widths and heights.
The `Auto` sized columns and rows of the default layout pick up these measurements and adjust themselves accordingly.

当然，图形大小默认不会适合这种排列，内容会被剪切：

\begin{examplefigure}{svg = true}

```julia
f = Figure()
for i in 1:5, j in 1:5
    Axis(f[i, j], width = 150, height = 150)
end
f
```

\end{examplefigure}

但像之前一样，我们可以通过调用 `resize_to_layout!` 来修正大小，确保不会发生剪切：

\begin{examplefigure}{svg = true}

```julia
set_theme!() # hide
resize_to_layout!(f)
f
```

\end{examplefigure}
