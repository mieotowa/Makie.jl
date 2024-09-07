# 如何在子图周围绘制框

If you want to show that several elements in a `Figure` belong together, you can do this by placing them all in a container.
如果您想表明 Figure 中的几个元素属于一组，可以通过将它们全部放入一个容器来实现这一目的。
诀窍是为每组对象使用嵌套的 `GridLayout`，并在相同位置放置一个 `Box`。
然后，设置 `alignmode = Outside(some_padding)` 可确保具有突出部分（如 `Axis`）的对象完全包含在包围框内。
Then the `alignmode = Outside(some_padding)` ensures that objects with protrusions sticking out, like `Axis`, are fully contained within the enclosing boxes.

\begin{examplefigure}{}

```julia
using CairoMakie
CairoMakie.activate!() # hide

f = Figure()

g1 = GridLayout(f[1, 1], alignmode = Outside(15))
g2 = GridLayout(f[1, 2], alignmode = Outside(15))
box1 = Box(f[1, 1], cornerradius = 10, color = (:tomato, 0.5), strokecolor = :transparent)
box2 = Box(f[1, 2], cornerradius = 10, color = (:teal, 0.5), strokecolor = :transparent)

# move the boxes back so the Axis background polys are in front of them
Makie.translate!(box1.blockscene, 0, 0, -100)
Makie.translate!(box2.blockscene, 0, 0, -100)

Axis(g1[1, 1], backgroundcolor = :white)
Axis(g1[2, 1], backgroundcolor = :white)

Axis(g2[1, 1], backgroundcolor = :white)
Axis(g2[1, 2], backgroundcolor = :white)
Axis(g2[2, 1:2], backgroundcolor = :white)

Label(f[0, :], "Two boxes indicate groups of axes that belong together")

f
```

\end{examplefigure}
