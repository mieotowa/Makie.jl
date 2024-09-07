# 文本框

`Textbox` 支持输入简单的单行字符串，同时可选配验证逻辑。

\begin{examplefigure}{}

```julia
using GLMakie
using CairoMakie # hide
CairoMakie.activate!() # hide

f = Figure()
Textbox(f[1, 1], placeholder = "Enter a string...")
Textbox(f[2, 1], width = 300)

f
```

\end{examplefigure}

## 验证

`validator` 属性与 `validate_textbox(string, validator)` 一起使用，判断当前字符串是否有效。 它可以是一个需要与完整字符串匹配的 `Regex`，或者是一个接受 `String` 作为输入并返回 `Bool` 的 `Function`。 如果验证器是一个类型 T（例如 `Float64`），验证将使用 `tryparse(T, string)`。 如果验证器未能通过，文本框将不允许提交当前输入的值。 It can be a `Regex` that needs to match the complete string, or a `Function` taking a `String` as input and returning a `Bool`. If the validator is a type T (for example `Float64`), validation will be `tryparse(T, string)`. The textbox will not allow submitting the currently entered value if the validator doesn't pass.

\begin{examplefigure}{}

```julia
using GLMakie
using CairoMakie # hide
CairoMakie.activate!() # hide

f = Figure()

tb = Textbox(f[2, 1], placeholder = "Enter a frequency",
    validator = Float64, tellwidth = false)

frequency = Observable(1.0)

on(tb.stored_string) do s
    frequency[] = parse(Float64, s)
end

xs = 0:0.01:10
sinecurve = @lift(sin.($frequency .* xs))

lines(f[1, 1], xs, sinecurve)

f
```

\end{examplefigure}

## 属性

\attrdocs{Textbox}
