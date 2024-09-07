# qqplot 和 qqnorm

{{doc qqplot}}
{{doc qqnorm}}

### 示例

测试 `xs` 和 `ys` 是否遵循相同的分布。

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide


xs = randn(100)
ys = randn(100)

qqplot(xs, ys, qqline = :identity)
```

\end{examplefigure}

测试 `ys` 是否服从正态分布。

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide


ys = 2 .* randn(100) .+ 3

qqnorm(ys, qqline = :fitrobust)
```

\end{examplefigure}
