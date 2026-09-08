# LaTeX

Makie 能够通过 [MathTeXEngine.jl](https://github.com/Kolaru/MathTeXEngine.jl/) 渲染来自 [LaTeXStrings.jl](https://github.com/stevengj/LaTeXStrings.jl) 包的 LaTeX 字符串。

虽然这个引擎响应足够用于 GLMakie，但它只支持 LaTeX 最常用的命令的子集。

## 使用 L-字符串

您可以将 `LaTeXString` 对象传递给几乎任何带有文本标签的对象。 它们使用 `L` 字符串宏前缀构建。
如果字符串不包含未转义的 `$`，则整个字符串被视为等式进行解释。 They are constructed using the `L` string macro prefix.
The whole string is interpreted as an equation if it doesn't contain an unescaped `$`.

```!
# hideall
using CairoMakie
```

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
CairoMakie.activate!() # hide

f = Figure(fontsize = 18)

Axis(f[1, 1],
    title = L"\forall \mathcal{X} \in \mathbb{R} \quad \frac{x + y}{\sin(k^2)}",
    xlabel = L"\sum_a^b{xy} + \mathscr{L}",
    ylabel = L"\sqrt{\frac{a}{b}} - \mathfrak{W}"
)

f
```

\end{examplefigure}

You can also mix math-mode and text-mode.
您还可以混合数学模式和文本模式。
对于[字符串插值](https://docs.julialang.org/en/v1/manual/strings/#string-interpolation) ，请使用 `%$` 替换 `$`：

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
CairoMakie.activate!() # hide

f = Figure(fontsize = 18)
t = "text"
Axis(f[1,1], title=L"Some %$(t) and some math: $\frac{2\alpha+1}{y}$")

f
```

\end{examplefigure}

## 统一字体

我们提供了一个 LaTeX 主题，以便轻松切换到所有文本的 LaTeX 默认字体。

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
CairoMakie.activate!() # hide

with_theme(theme_latexfonts()) do
    fig = Figure()
    Label(fig[1, 1], "A standard Label", tellwidth = false)
    Label(fig[2, 1], L"A LaTeXString with a small formula $x^2$", tellwidth = false)
    Axis(fig[3, 1], title = "An axis with matching font for the tick labels")
    fig
end
```

\end{examplefigure}
