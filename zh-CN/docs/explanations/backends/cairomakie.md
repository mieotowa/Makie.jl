# CairoMakie

[CairoMakie](https://github.com/MakieOrg/Makie.jl/tree/master/CairoMakie) 利用 Cairo.jl 来绘制矢量图形到 SVG 和 PDF 格式。
如果你希望为出版物生成最高质量的图表，应当选择使用它，因为 GL 后端的渲染过程基于位图，更侧重于速度而非像素级别的完美呈现。
You should use it if you want to achieve the highest-quality plots for publications, as the rendering process of the GL backends works via bitmaps and is geared more towards speed than pixel-perfection.

## 激活 CairoMakie 与 screen 配置

通过调用 `CairoMakie.activate!()` 函数并设置以下选项来激活该后端：

```julia:docs
# hideall
using CairoMakie, Markdown
println("~~~")
println(Markdown.html(@doc CairoMakie.activate!))
println("~~~")
```

\textoutput{docs}

#### 内联图表类型

你可以通过 `activate!` 函数来设定在诸如 VSCode、Pluto.jl 或其他环境中内嵌显示图表的类型：

```julia
CairoMakie.activate!(type = "png")
CairoMakie.activate!(type = "svg")
```

#### Z 轴顺序

CairoMakie as a 2D engine has no concept of z-clipping, therefore its 3D capabilities are quite limited.
The z-values of 3D plots will have no effect and will be projected flat onto the canvas.
Z-layering is approximated by sorting all plot objects by their z translation value before drawing, after that by parent scene and then insertion order.
Therefore, if you want to draw something on top of something else, but it ends up below, try translating it forward via `translate!(obj, 0, 0, some_positive_z_value)`.

#### 选择性栅格化

通过设置图表的 `rasterize` 属性，即使保存至矢量后端，你也可以指示 CairoMakie 对该图表进行栅格化处理。  这对于大型网格、表面，甚至是不规则网格上的热力图非常有用。  This can be very useful for large meshes, surfaces or even heatmaps if on an irregular grid.

假设您有一个`Plot`对象`plt`，您可以设置`plt.rasterize = true`进行简单的光栅化，或者您可以设置`plt.rasterize = scale::Int`，其中`scale`表示图像表面的缩放因子。

例如，如果场景的尺寸为 `(800, 600)`，设置 `scale=2` 会使矢量图形中嵌入的栅格化图像具有 `(1600, 1200)` 的分辨率。

要取消栅格化，只需设置 `plt.rasterize = false`。

示例：

\begin{examplefigure}{svg=true}

```julia
using CairoMakie
CairoMakie.activate!() # hide
v = rand(10,2)
scatter(v[:,1], v[:,2], rasterize = true, markersize = 30.0)
```

\end{examplefigure}

当你放大观察时，会发现图像像素化严重——这是因为栅格化密度设置为每 pt 1 个像素。  将 `rasterize` 设置为 10 可显著提升平滑度：  Setting `rasterize=10` makes this a lot smoother:

\begin{examplefigure}{svg=true}

```julia
using CairoMakie
CairoMakie.activate!() # hide
v = rand(10,2)
scatter(v[:,1], v[:,2], rasterize = 10, markersize = 30.0)
```

\end{examplefigure}
