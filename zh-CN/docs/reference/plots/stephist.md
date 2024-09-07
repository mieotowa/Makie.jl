# stephist

{{doc stephist}}

### 示例

\begin{examplefigure}{}

```julia
using GLMakie
GLMakie.activate!() # hide


data = randn(1000)

f = Figure()
stephist(f[1, 1], data, bins = 10)
stephist(f[1, 2], data, bins = 20, color = :red, strokewidth = 1, strokecolor = :black)
stephist(f[2, 1], data, bins = [-5, -2, -1, 0, 1, 2, 5], color = :gray)
stephist(f[2, 2], data, normalization = :pdf)
f
```

\end{examplefigure}

有关更多示例，请参见 `hist`。
