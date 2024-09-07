# 横杆（crossbar）

{{doc crossbar}}

### 示例

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide


xs = [1, 1, 2, 2, 3, 3]
ys = rand(6)
ymins = ys .- 1
ymaxs = ys .+ 1
dodge = [1, 2, 1, 2, 1, 2]

crossbar(xs, ys, ymins, ymaxs, dodge = dodge, show_notch = true)
```

\end{examplefigure}
