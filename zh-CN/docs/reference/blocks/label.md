# 标签（Label）

Label 是一个矩形边界内的文本。
`halign` 和 `valign` 属性始终与未旋转的水平和垂直方向相关。
这与 `text` 不同，在 text 中对齐是相对于文字流动方向的。
The `halign` and `valign` attributes always refer to unrotated horizontal and vertical.
This is different from `text`, where alignment is relative to text flow direction.

Label 的大小是已知的，因此如果将 `tellwidth` 和 `tellheight` 设置为 `true`（默认值），带有 `Auto` 列或行尺寸的 GridLayout 可以缩小以适应 Label 大小。

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
CairoMakie.activate!() # hide

fig = Figure()

fig[1:2, 1:3] = [Axis(fig) for _ in 1:6]

supertitle = Label(fig[0, :], "Six plots", fontsize = 30)

sideinfo = Label(fig[1:2, 0], "This text is vertical", rotation = pi/2)

fig
```

\end{examplefigure}

Label 的对齐方式和行距可以像处理普通文本一样进行控制。

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
CairoMakie.activate!() # hide

f = Figure()

Label(f[1, 1],
    "Multiline label\nwith\njustification = :left\nand\nlineheight = 0.9",
    justification = :left,
    lineheight = 0.9
)
Label(f[1, 2],
    "Multiline label\nwith\njustification = :center\nand\nlineheight = 1.1",
    justification = :center,
    lineheight = 1.1,
    color = :dodgerblue,
)
Label(f[1, 3],
    "Multiline label\nwith\njustification = :right\nand\nlineheight = 1.3",
    justification = :right,
    lineheight = 1.3,
    color = :firebrick
)

f
```

\end{examplefigure}

## 属性

\attrdocs{Label}
