# 线段（linesegments）

{{doc linesegments}}

## 示例

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide


f = Figure()
Axis(f[1, 1])

xs = 1:0.2:10
ys = sin.(xs)

linesegments!(xs, ys)
linesegments!(xs, ys .- 1, linewidth = 5)
linesegments!(xs, ys .- 2, linewidth = 5, color = LinRange(1, 5, length(xs)))

f
```

\end{examplefigure}

### 处理GLMakie中的线条轮廓伪影

In GLMakie 3D line plots can generate outline artifacts depending on the order line segments are rendered in.
Currently there are a few ways to mitigate this problem, but they all come at a cost:

- `fxaa = true` 会禁用线段的原生抗锯齿功能，转而使用 FXAA。 这会导致线条细节减少。 This results in less detailed lines.
- `transparency = true` 会在一定程度上禁用深度测试，从而消除所有线条的伪影。 然而，这样一来线条总会带有一定的透明度。 However with this lines will always have some level of transparency.
- `overdraw = true` 会完全禁用（读取和写入）该图的深度测试，去除伪影。 但这会改变线段的 Z 顺序，使得后期渲染的图形出现在 linesegments 图之上。 This will however change the z-order of line segments and allow plots rendered later to show up on top of the linesegments plot.

\begin{examplefigure}{}

```julia
using GLMakie
GLMakie.activate!() # hide

ps = rand(Point3f, 500)
cs = rand(500)
f = Figure(size = (600, 650))
Label(f[1, 1], "base", tellwidth = false)
linesegments(f[2, 1], ps, color = cs, markersize = 20, fxaa = false)
Label(f[1, 2], "fxaa = true", tellwidth = false)
linesegments(f[2, 2], ps, color = cs, markersize = 20, fxaa = true)
Label(f[3, 1], "transparency = true", tellwidth = false)
linesegments(f[4, 1], ps, color = cs, markersize = 20, transparency = true)
Label(f[3, 2], "overdraw = true", tellwidth = false)
linesegments(f[4, 2], ps, color = cs, markersize = 20, overdraw = true)
f
```

\end{examplefigure}
