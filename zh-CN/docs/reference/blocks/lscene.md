# LScene

如果您需要在布局中使用一个常规的 Makie 场景（如用于3D绘图），目前您需要使用 `LScene`。 它只是对常规 `Scene` 的一个封装，使其成为可布局的块（block）。 通过 `scene` 字段即可访问底层的 Scene。
您可以直接向 `LScene` 绘图。 It's just a wrapper around the normal `Scene` that
makes it block. The underlying Scene is accessible via the `scene` field.
You can plot into the `LScene` directly, though.

您可以通过 `scenekw` 关键字将关键字参数传递给底层 `Scene` 对象。
目前，可能需要显式传递一些属性以确保它们不会从主场景继承。
要查看适用的参数，请参阅 [scene docs](/explanations/scenes)
Currently, it can be necessary to pass a couple of attributes explicitly to make sure they are not inherited from the main scene.
To see what parameters are applicable, have a look at the [scene docs](/explanations/scenes)

\begin{examplefigure}{}

```julia
using GLMakie
GLMakie.activate!()

fig = Figure()
pl = PointLight(Point3f(0), RGBf(20, 20, 20))
al = AmbientLight(RGBf(0.2, 0.2, 0.2))
lscene = LScene(fig[1, 1], show_axis=false, scenekw = (lights = [pl, al], backgroundcolor=:black, clear=true))
# now you can plot into lscene like you're used to
p = meshscatter!(lscene, randn(300, 3), color=:gray)
fig
```

\end{examplefigure}

## 属性

\attrdocs{LScene}
