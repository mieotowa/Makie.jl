# Backends

Makie是定义所有绘图功能的前端包。
每个后端都会重新导出它，因此您无需特别安装或导入它。
It is reexported by every backend, so you don't have to specifically install or import it.

有四个后端具体实现了 Makie 中定义的所有抽象渲染能力：

| 包名                                                    | 描述                                               |
| :---------------------------------------------------- | :----------------------------------------------- |
| [`GLMakie.jl`](/explanations/backends/glmakie/)       | 基于 GPU、在独立 `GLFW.jl` 窗口中进行交互式 2D 和 3D 绘图的后端。     |
| [`CairoMakie.jl`](/explanations/backends/cairomakie/) | 基于 `Cairo.jl`、非交互式的 2D（及部分 3D）后端，适用于生成出版质量的矢量图形。 |
| [`WGLMakie.jl`](/explanations/backends/wglmakie/)     | 基于 WebGL 的交互式 2D 和 3D 绘图后端，可在浏览器内运行。             |
| [`RPRMakie.jl`](/explanations/backends/rprmakie/)     | 一个实验性的光线追踪后端。                                    |

### 激活后端

您可以通过 `using` 相应的包并调用其 `activate!` 函数来激活任何后端。

以WGLMakie为例：

```julia
using WGLMakie
WGLMakie.activate!()
```

每个后端的 `activate!` 函数可选地接受关键字参数（称为 `screen_config...`），用于控制后端的各种方面。
例如，要激活GLMakie后端并将其配置为生成具有自定义标题且无抗锯齿的窗口：
For example, to activate the GLMakie backend and set it up to produce windows with a custom title and no anti-aliasing:

```julia
using GLMakie
GLMakie.activate!(title = "Custom title", fxaa = false)
```

各后端所接受的关键字参数在其表格上方链接的特定后端文档页面中有详细列出。
