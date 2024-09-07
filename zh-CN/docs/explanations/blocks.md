# 块（Blocks）

`Blocks` 是可以添加到 `Figure` 或 `Scene` 中，并由 `GridLayout` 控制其位置和大小的对象。 作为一种抽象类型，`Block` 本身并无实际内容。
`Figure` 内部自带 `GridLayout`，因此提供了向其中添加块的简化语法。
如果您想要直接操作裸露的 `Scene`，则可以将 `GridLayout` 附加到其像素区域上。 In of itself, a `Block` is an abstract type.
A `Figure` has its own internal `GridLayout` and therefore offers simplified syntax for adding blocks to it.
If you want to work with a bare `Scene`, you can attach a `GridLayout` to its pixel area.

!!! note
A layout only controls an object's position or bounding box.
注意 布局仅控制对象的位置或边界框。
`Block` 可以受 Figure 的 GridLayout 控制，但不必作为视觉元素添加到该 Figure 中。
`Block` 也可以不位于任何 GridLayout 内而直接添加到 Scene 中。
A `Block` can also be added to a Scene without being inside any GridLayout, if you specify the bounding box yourself.

## 添加至 `Figure`

以下是向 Figure 添加一个 `Block`（本例为 `Axis`）的一种方法：

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide


f = Figure()
ax = Axis(f[1, 1])
f
```

\end{examplefigure}

## 直接指定边界框

Sometimes you just want to place a `Block` in a specific location, without it being controlled by a dynamic layout.
You can do this by setting the `bbox` parameter, which is usually controlled by the layout, manually.
有时您可能希望在不受动态布局控制的情况下将 `Block` 放置在特定位置。
为此，您可以手动设置通常由布局控制的 `bbox` 参数。
边界框应为一个 2D `Rect`，如果计划动态更改，也可以是 Observable 类型。
函数 `BBox` 创建一个 `Rect2f`，但不同于传递原点和宽高，您直接传递左、右、下、上边界。
The function `BBox` creates an `Rect2f`, but instead of passing origin and widths, you pass left, right, bottom and top boundaries directly.

以下示例展示了两个轴被手动放置的情况：

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide


f = Figure()
Axis(f, bbox = BBox(100, 300, 100, 500), title = "Axis 1")
Axis(f, bbox = BBox(400, 700, 200, 400), title = "Axis 2")
f
```

\end{examplefigure}

## 删除块

要从布局及其所属的 Figure 或 Scene 中移除 Blocks，请使用 `delete!(block)`。
