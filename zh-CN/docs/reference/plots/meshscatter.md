# 网格散点（meshscatter）

{{doc meshscatter}}

## 示例

\begin{examplefigure}{}

```julia
using GLMakie
GLMakie.activate!() # hide


xs = cos.(1:0.5:20)
ys = sin.(1:0.5:20)
zs = LinRange(0, 3, length(xs))

meshscatter(xs, ys, zs, markersize = 0.1, color = zs)
```

\end{examplefigure}
