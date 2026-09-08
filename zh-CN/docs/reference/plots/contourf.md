# contourf

{{doc contourf}}

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
using DelimitedFiles
CairoMakie.activate!() # hide


volcano = readdlm(Makie.assetpath("volcano.csv"), ',', Float64)

f = Figure()
Axis(f[1, 1])

co = contourf!(volcano, levels = 10)

Colorbar(f[1, 2], co)

f
```

\end{examplefigure}

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
using DelimitedFiles
CairoMakie.activate!() # hide


volcano = readdlm(Makie.assetpath("volcano.csv"), ',', Float64)

f = Figure()
ax = Axis(f[1, 1])

co = contourf!(volcano,
    levels = range(100, 180, length = 10),
    extendlow = :cyan, extendhigh = :magenta)

tightlimits!(ax)

Colorbar(f[1, 2], co)

f
```

\end{examplefigure}

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
using DelimitedFiles
CairoMakie.activate!() # hide


volcano = readdlm(Makie.assetpath("volcano.csv"), ',', Float64)

f = Figure()
ax = Axis(f[1, 1])

co = contourf!(volcano,
    levels = range(100, 180, length = 10),
    extendlow = :auto, extendhigh = :auto)

tightlimits!(ax)

Colorbar(f[1, 2], co)

f
```

\end{examplefigure}

#### 相对模式

Sometimes it's beneficial to drop one part of the range of values, usually towards the outer boundary.
有时，为了突出中间部分的细节，会希望忽略数据值范围的某个端点（通常是边界）。
此时，无需手动指定包含的等级，而是可以将 `mode` 属性设置为 `:relative`，并以当前最小值和最大值为基础，使用 0 到 1 的比例来指定等级。

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
using DelimitedFiles
CairoMakie.activate!() # hide


volcano = readdlm(Makie.assetpath("volcano.csv"), ',', Float64)

f = Figure(size = (800, 400))

Axis(f[1, 1], title = "Relative mode, drop lowest 30%")
contourf!(volcano, levels = 0.3:0.1:1, mode = :relative)

Axis(f[1, 2], title = "Normal mode")
contourf!(volcano, levels = 10)

f
```

\end{examplefigure}
