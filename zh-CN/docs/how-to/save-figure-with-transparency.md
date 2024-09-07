# 如何保存具有透明背景的 `Figure`

## CairoMakie

在 CairoMakie 中，将背景颜色设置为 `:transparent`（转换为 `RGBAf(0, 0, 0, 0)`），即可获得全透明背景。
以下示例中，我使用了半透明蓝色，因为在白色页面上透明背景与常规白色难以区分。
In the following examples, I use a partially transparent blue because a transparent background is indistinguishable from the usual white on a white page.

\begin{examplefigure}{svg = true}

```julia
using CairoMakie
CairoMakie.activate!() # hide

f = Figure(backgroundcolor = (:blue, 0.4))
Axis(f[1, 1], backgroundcolor = (:tomato, 0.5))
f
```

\end{examplefigure}

## GLMakie

出于技术原因，GLMakie 的颜色缓冲区不包含 alpha 组件：

\begin{examplefigure}{}

```julia
using GLMakie
GLMakie.activate!() # hide
Makie.inline!(true) # hide

f = Figure(backgroundcolor = (:blue, 0.4))
Axis(f[1, 1], backgroundcolor = (:tomato, 0.5))
f
```

\end{examplefigure}

通过以下技巧，您仍然可以保存具有透明背景的图像。
其原理是设置两种不同的背景颜色，并通过计算两者之间的差异来得出带 alpha 通道的前景色。
It works by setting two different background colors and calculating the foreground color with alpha from the difference.

```julia:transparent-glmakie
using GLMakie
GLMakie.activate!() # hide
Makie.inline!(true) # hide

function calculate_rgba(rgb1, rgb2, rgba_bg)::RGBAf
    rgb1 == rgb2 && return RGBAf(rgb1.r, rgb1.g, rgb1.b, 1)
    c1 = Float64.((rgb1.r, rgb1.g, rgb1.b))
    c2 = Float64.((rgb2.r, rgb2.g, rgb2.b))
    alphas_fg = 1 .+ c1 .- c2
    alpha_fg = clamp(sum(alphas_fg) / 3, 0, 1)
    alpha_fg == 0 && return rgba_bg
    rgb_fg = clamp.((c1 ./ alpha_fg), 0, 1)
    rgb_bg = Float64.((rgba_bg.r, rgba_bg.g, rgba_bg.b))
    alpha_final = alpha_fg + (1 - alpha_fg) * rgba_bg.alpha
    rgb_final = @. 1 / alpha_final * (alpha_fg * rgb_fg + (1 - alpha_fg) * rgba_bg.alpha * rgb_bg)
    return RGBAf(rgb_final..., alpha_final)
end

function alpha_colorbuffer(figure)
    scene = figure.scene
    bg = scene.backgroundcolor[]
    scene.backgroundcolor[] = RGBAf(0, 0, 0, 1)
    b1 = copy(colorbuffer(scene))
    scene.backgroundcolor[] = RGBAf(1, 1, 1, 1)
    b2 = colorbuffer(scene)
    scene.backgroundcolor[] = bg
    return map(b1, b2) do b1, b2
        calculate_rgba(b1, b2, bg)
    end
end

f = Figure(backgroundcolor = (:blue, 0.4))
Axis(f[1, 1], backgroundcolor = (:tomato, 0.5))
f

save(joinpath(@OUTPUT, "transparent.png"), alpha_colorbuffer(f)) # hide
save("transparent.png", alpha_colorbuffer(f))
```

\fig{transparent.png}
