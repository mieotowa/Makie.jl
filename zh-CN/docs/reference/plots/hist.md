# hist

{{doc hist}}

### 示例

\begin{examplefigure}{}

```julia
using GLMakie
GLMakie.activate!() # hide


data = randn(1000)

f = Figure()
hist(f[1, 1], data, bins = 10)
hist(f[1, 2], data, bins = 20, color = :red, strokewidth = 1, strokecolor = :black)
hist(f[2, 1], data, bins = [-5, -2, -1, 0, 1, 2, 5], color = :gray)
hist(f[2, 2], data, normalization = :pdf)
f
```

\end{examplefigure}

#### 带标签的直方图

您可以使用与 \apilink{barplot} 相同的所有参数： \begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide

data = randn(1000)

hist(data, normalization = :pdf, bar_labels = :values,
     label_formatter=x-> round(x, digits=2), label_size = 15,
     strokewidth = 0.5, strokecolor = (:black, 0.5), color = :values)
```

\end{examplefigure}

#### 移动直方图

With `scale_to`, and `offset`, one can put multiple histograms into the same plot.
Note, that offset automatically sets fillto, to move the whole barplot.
通过 `scale_to` 和 `offset` 参数，您可以在同一幅图中放置多个直方图。
请注意，offset 参数会自动设置 fillto，以移动整个条形图。
此外，您还可以使用负值 `scale_to` 使直方图翻转，或者使用 `scale_to=:flip` 来改变条形的方向而不改变其高度。

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide

fig = Figure()
ax = Axis(fig[1, 1])
for i in 1:5
     hist!(ax, randn(1000), scale_to=-0.6, offset=i, direction=:x)
end
fig
```

\end{examplefigure}

#### 使用统计权重

\begin{examplefigure}{}

```julia
using CairoMakie, Distributions
CairoMakie.activate!() # hide


N = 100_000
x = rand(Uniform(-5, 5), N)

w = pdf.(Normal(), x)

fig = Figure()
hist(fig[1,1], x)
hist(fig[1,2], x, weights = w)

fig
```

\end{examplefigure}
