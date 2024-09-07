# 滑块（Slider）

A simple slider without a label. You can create a label using a `Label` object,
for example. You need to specify a range that constrains the slider's possible values.

The currently selected value is in the attribute `value`.
当前选定的值存储在属性 `value` 中。
不要手动更改这个值，而是使用函数 `set_close_to!(slider, value)`
来确保值确实存在于 `range` 属性内。
This is necessary to ensure the value is actually present in the `range` attribute.

双击滑块可将其重置到（大约）startvalue 中存储的值。

如果设置属性 `snap = false`，滑块在拖动过程中会连续移动，并仅在释放鼠标时跳至最近的有效值。

\begin{examplefigure}{}

```julia
using GLMakie
GLMakie.activate!() # hide
fig = Figure()

ax = Axis(fig[1, 1])

sl_x = Slider(fig[2, 1], range = 0:0.01:10, startvalue = 3)
sl_y = Slider(fig[1, 2], range = 0:0.01:10, horizontal = false, startvalue = 6)

point = lift(sl_x.value, sl_y.value) do x, y
    Point2f(x, y)
end

scatter!(point, color = :red, markersize = 20)

limits!(ax, 0, 10, 0, 10)

fig
```

\end{examplefigure}

## 带标签的滑块和网格

函数 \apilink{labelslider!} 和 \apilink{labelslidergrid!} 已弃用，请改用 \apilink{SliderGrid}。

## 属性

\attrdocs{Slider}
