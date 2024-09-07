# RPRMakie

RPRMakie 是一个实验性的光线追踪后端，使用 AMD 开发的 [RadeonProRender](https://radeon-pro.github.io/RadeonProRenderDocs/en/index.html) 技术。
尽管由 AMD 制作并针对 Radeon GPU 优化，它同样适用于使用 OpenCL 的 NVIDIA 和 Intel GPU。
此外，RPRMakie 还能在 CPU 上运行，并且具备混合模式，能够同时利用 GPU 和 CPU 合力渲染图像。
While it's created by AMD and tailored to Radeon GPUs, it still works just as well for NVidia and Intel GPUs using OpenCL.
It also works on the CPU and even has a hybrid modus to use GPUs and CPUs in tandem to render images.

目前，RadeonProRender 仅支持 Windows 和 Linux 系统，在 macOS 上尝试创建最基础的上下文时会崩溃。 如果您使用的是 macOS 并擅长调试段错误（segfault），请帮助我们进行调试： If you're on OSX and good at debugging segfaults, help us by debugging:

```julia
using RadeonProRender
RadeonProRender.Context()
```

要在配备 M 系列芯片的 Mac 上使用 RPRMakie，当前需要使用 x86 构建的 Julia（而非 ARM 构建，可能需要手动下载）。  RadeonProRender 尚未发布针对 M 系列处理器 ARM 架构的二进制文件。  RadeonProRender does not distribute binaries built for the ARM architecture of the M-series processors yet.

## 激活 RPRMakie 与 screen 配置

通过调用 `RPRMakie.activate!()` 函数并设置以下选项来激活该后端：

```julia:docs
# hideall
using RPRMakie, Markdown
println("~~~")
println(Markdown.html(@doc RPRMakie.activate!))
println("~~~")
```

\textoutput{docs}

由于 RPRMakie 是一个非常独特且仍在实验阶段的后端，在使用时需要注意以下几个要点：

```julia
fig = Figure(); # RPRMakie 当前不支持显示 Figures，因为它只支持物理三维相机
radiance = 10000
# 对于光线追踪来说，光源至关重要，
# 因此大多数示例会使用额外的灯光和环境光源。
# 注意，RPRMakie 是当前唯一支持多光源和 EnvironmentLights 的后端
lights = [
    EnvironmentLight(0.5, Makie.FileIO.load(RPR.assetpath("studio026.exr"))),
    PointLight(Vec3f(0, 0, 20), RGBf(radiance, radiance, radiance))
]

# 目前仅支持 LScene，
# 因为其他投影方式与 RPR 中物理精确的相机不匹配
ax = LScene(fig[1, 1]; show_axis = false, scenekw=(lights=lights,))
# 由于 RPRMakie 当前还不支持文字（正在开发中！），
# 无法显示三维坐标轴

# 创建材质时需要访问 RPR 上下文。
# 注意，如果您手动创建了 Screen，则不应再对场景或 fig 调用 display，因为那样会创建一个新的 RPR 上下文，在其中手动创建的上下文中的资源无效。由于 RPR 错误处理较差，这通常会导致段错误（Segfaults）。
# 请参阅下文如何使用手动创建的上下文渲染图像
screen = RPRMakie.Screen(ax.scene; iterations=10, plugin=RPR.Northstar)
matsys = screen.matsys
context = screen.context
# 您可以使用 RPR 中的多种材质。
# 注意，此 API 可能在未来发生变化，以适应独立于后端的表示方式
# 或至少不再需要直接访问 RPR 上下文
mat = RPR.Chrome(matsys)
# 材质属性是 RPRMakie 特有的，被其他后端忽略。这一情况未来可能会有所变化
mesh!(ax, Sphere(Point3f(0), 1), material=mat)

# 将 Makie 场景转化为图像有三种主要方式：
# 获取 Screen 的颜色缓冲区。Screen 对 `image\png` MIME 类型的 `show` 方法进行了重载，因此应该能在 IJulia、Jupyter 或 VSCode 中显示
image = colorbuffer(screen)::Matrix{RGB{N0f8}}
# 使用 RPR 替换特定（子）LScene，并在 RPRMakie 中交互式地显示整个场景
using RPRMakie
refres = Observable(nothing) # 可选的触发刷新的 Observable
RPRMakie.activate!(); display(fig) # 确保首先在 RPRMakie 中显示场景
# 用交互式 RPR 输出替换场景。有关此功能的更多信息，请参阅 RPRMakie 互操作示例
context, task = RPRMakie.replace_scene_rpr!(ax.scene, screen; refresh=refresh)
# 如果不手动创建 Screen 来创建自定义材质，
# display(ax.scene), show(io, MIME"image/png", ax.scene), save("rpr.png", ax.scene)
# 应该像使用其他后端一样正常工作。
# 注意，只有来自 LScene 的场景可以直接显示，但很快 `display(fig)` 也应该能工作。
```

There are several examples showing different aspects of how to use RPRMakie.
RPRMakie 提供了一系列示例，展示了如何使用该后端的不同方面。
这些示例位于 [RPRMakie/examples](https://github.com/MakieOrg/Makie.jl/tree/master/RPRMakie/examples)

## MaterialX 和预定义材质（materials.jl）

RadeonProRender 中有几种预定义的材质可供使用。
RPR还支持 [MaterialX](https://www.materialx.org/) 标准，以加载各种预定义材质。 请确保在 `MaterialX` 中使用 Northstar 后端。
RPR also supports the [MaterialX](https://www.materialx.org/) standard to load a wide range of predefined Materials. Make sure to use the Northstar backend for `MaterialX`.

```
<input id="hidecode5" class="hidecode" type="checkbox">
```

```julia
using GeometryBasics, RPRMakie
using Colors, FileIO
using Colors: N0f8

radiance = 500
lights = [EnvironmentLight(1.0, load(RPR.assetpath("studio026.exr"))),
            PointLight(Vec3f(10), RGBf(radiance, radiance, radiance * 1.1))]
fig = Figure(; size=(1500, 700));
ax = LScene(fig[1, 1]; show_axis=false, scenekw=(; lights=lights))
screen = RPRMakie.Screen(ax.scene; plugin=RPR.Northstar, iterations=400)

matsys = screen.matsys
emissive = RPR.EmissiveMaterial(matsys)
diffuse = RPR.DiffuseMaterial(matsys)
glass = RPR.Glass(matsys)
plastic = RPR.Plastic(matsys)
chrome = RPR.Chrome(matsys)
dielectric = RPR.DielectricBrdfX(matsys)
gold = RPR.SurfaceGoldX(matsys)

materials = [glass chrome;
                gold dielectric;
                emissive plastic]

mesh!(ax, load(Makie.assetpath("matball_floor.obj")); color=:white)
palette = reshape(Makie.default_palettes.color[][1:6], size(materials))

for i in CartesianIndices(materials)
    x, y = Tuple(i)
    mat = materials[i]
    mplot = if mat === emissive
        matball!(ax, diffuse; inner=emissive, color=nothing)
    else
        matball!(ax, mat; color=nothing)
    end
    v = Vec3f(((x, y) .- (0.5 .* size(materials)) .- 0.5)..., 0)
    translate!(mplot, 0.9 .* (v .- Vec3f(0, 3, 0)))
end
cam = cameracontrols(ax.scene)
cam.eyeposition[] = Vec3f(-0.3, -5.5, 0.9)
cam.lookat[] = Vec3f(0.5, 0, -0.5)
cam.upvector[] = Vec3f(0, 0, 1)
cam.fov[] = 35
emissive.color = Vec3f(4, 2, 2)
image = colorbuffer(screen)
save("materials.png", image)
```

```
<label for="hidecode5" class="hidecode"></label>
```

```
<img src="/assets/materials.png">
```

## 高级自定义材质（earth_topography.jl）

```
<input id="hidecode4" class="hidecode" type="checkbox">
```

```julia
using NCDatasets, ColorSchemes, RPRMakie
using ImageShow, FileIO

# Taken from https://lazarusa.github.io/BeautifulMakie/GeoPlots/topography/
cmap = dataset = Dataset(joinpath(@__DIR__, "ETOPO1_halfdegree.nc"))
lon = dataset["lon"][:]
lat = dataset["lat"][:]
data = Float32.(dataset["ETOPO1avg"][:, :])

function glow_material(data_normed)
    emission_weight = map(data_normed) do i
        return Float32(i < 0.7 ? 0.0 : i)
    end
    emission_color = map(data_normed) do i
        em = i * 2
        return RGBf(em * 2.0, em * 0.4, em * 0.3)
    end

    return (
        reflection_weight = 1,
        reflection_color = RGBf(0.5, 0.5, 1.0),
        reflection_metalness = 0,
        reflection_ior = 1.4,
        diffuse_weight = 1,
        emission_weight = emission_weight',
        emission_color = emission_color',
    )
end

RPRMakie.activate!(iterations=32, plugin=RPR.Northstar)
fig = Figure(; size=(2000, 800))
radiance = 30000
lights = [EnvironmentLight(1.0, load(RPR.assetpath("studio026.exr"))),
            PointLight(Vec3f(0, 100, 100), RGBf(radiance, radiance, radiance))]

ax = LScene(fig[1, 1]; show_axis=false, scenekw=(lights=lights,))

mini, maxi = extrema(data)
data_normed = ((data .- mini) ./ (maxi - mini))

material = glow_material(data_normed)

pltobj = surface!(ax, lon, lat, data_normed .* 20;
                    material=material, colormap=[:black, :white, :brown],
                    colorrange=(0.2, 0.8) .* 20)
# Set the camera to a nice angle
cam = cameracontrols(ax.scene)
cam.eyeposition[] = Vec3f(3, -300, 300)
cam.lookat[] = Vec3f(0)
cam.upvector[] = Vec3f(0, 0, 1)
cam.fov[] = 23

save("topographie.png", ax.scene)
```

```
<label for="hidecode4" class="hidecode"></label>
```

```
<img src="/assets/topographie.png">
```

## RPRMakie 互操作（opengl_interop.jl）

RPRMakie 目前不支持布局和子场景，但您可以用 RPR 渲染的交互式窗口替换单个场景。
这对于在光线追踪场景旁边显示 2D 图形和交互式 UI 元素，并交互地调整相机和材质参数特别方便。
This is especially handy, to show 2d graphics and interactive UI elements next to a ray traced scene and interactively tune camera and material parameters.

```
<input id="hidecode3" class="hidecode" type="checkbox">
```

```julia
using RPRMakie, GeometryBasics, RPRMakie, RadeonProRender
using Colors, FileIO
using Colors: N0f8

f = (u, v) -> cos(v) * (6 - (5 / 4 + sin(3 * u)) * sin(u - 3 * v))
g = (u, v) -> sin(v) * (6 - (5 / 4 + sin(3 * u)) * sin(u - 3 * v))
h = (u, v) -> -cos(u - 3 * v) * (5 / 4 + sin(3 * u));
u = range(0; stop=2π, length=150)
v = range(0; stop=2π, length=150)
radiance = 500
lights = [EnvironmentLight(1.0, load(RPR.assetpath("studio026.exr"))),
          PointLight(Vec3f(10), RGBf(radiance, radiance, radiance * 1.1))]

fig = Figure(; size=(1500, 1000))
ax = LScene(fig[1, 1]; show_axis=false, scenekw=(; lights=lights))
screen = RPRMakie.Screen(size(ax.scene); plugin=RPR.Tahoe)
material = RPR.UberMaterial(screen.matsys)

surface!(ax, f.(u, v'), g.(u, v'), h.(u, v'); ambient=Vec3f(0.5), diffuse=Vec3f(1), specular=0.5,
         colormap=:balance, material=material)

function Input(fig, val::RGB)
    hue = Slider(fig; range=1:380, width=200)
    lightness = Slider(fig; range=LinRange(0, 1, 100), width=200)
    labels = [Label(fig, "hue"; halign=:left), Label(fig, "light"; halign=:left)]
    layout = grid!(hcat(labels, [hue, lightness]))
    hsl = HSL(val)
    set_close_to!(hue, hsl.h)
    set_close_to!(lightness, hsl.l)
    color = map((h, l) -> RGB(HSL(h, 0.9, l)), hue.value, lightness.value)
    return color, layout
end

function Input(fig, val::Vec4)
    s = Slider(fig; range=LinRange(0, 1, 100), width=200)
    set_close_to!(s, first(val))
    return map(x -> Vec4f(x), s.value), s
end

function Input(fig, val::Bool)
    toggle = Toggle(fig; active=val)
    return toggle.active, toggle
end

sliders = (reflection_color=Input(fig, RGB(0, 0, 0)), reflection_weight=Input(fig, Vec4(0)),
           reflection_roughness=Input(fig, Vec4(0)), reflection_anisotropy=Input(fig, Vec4(0)),
           reflection_anisotropy_rotation=Input(fig, Vec4(0)), reflection_mode=Input(fig, Vec4(0)),
           reflection_ior=Input(fig, Vec4(0)), reflection_metalness=Input(fig, Vec4(0)),
           refraction_color=Input(fig, RGB(0, 0, 0)), refraction_weight=Input(fig, Vec4(0)),
           refraction_roughness=Input(fig, Vec4(0)), refraction_ior=Input(fig, Vec4(0)),
           refraction_absorption_color=Input(fig, RGB(0, 0, 0)),
           refraction_absorption_distance=Input(fig, Vec4(0)), refraction_caustics=Input(fig, true),
           sss_scatter_color=Input(fig, RGB(0, 0, 0)), sss_scatter_distance=Input(fig, Vec4(0)),
           sss_scatter_direction=Input(fig, Vec4(0)), sss_weight=Input(fig, Vec4(0)),
           sss_multiscatter=Input(fig, false), backscatter_weight=Input(fig, Vec4(0)),
           backscatter_color=Input(fig, RGB(0, 0, 0)))

labels = []
inputs = []
refresh = Observable(nothing)
for (key, (obs, input)) in pairs(sliders)
    push!(labels, Label(fig, string(key); align=:left))
    push!(inputs, input)
    on(obs) do value
        setproperty!(material, key, value)
        return notify(refresh)
    end
end

fig[1, 2] = grid!(hcat(labels, inputs); width=500)
RPRMakie.activate!()

cam = cameracontrols(ax.scene)
cam.eyeposition[] = Vec3f(22, 0, 17)
cam.lookat[] = Vec3f(0, 0, -1)
cam.upvector[] = Vec3f(0, 0, 1)
cam.fov[] = 30

display(fig)

context, task = RPRMakie.replace_scene_rpr!(ax.scene, screen; refresh=refresh)

# Change light parameters interactively
begin
    lights[1].intensity[] = 1.5
    lights[2].radiance[] = RGBf(1000, 1000, 1000)
    lights[2].position[] = Vec3f(3, 10, 10)
    notify(refresh)
end
```

```
<label for="hidecode3" class="hidecode"></label>
```

```
<video mute autoplay controls src="/assets/opengl_interop.mp4">
</video>
```

## 动画（lego.jl）

并非所有对象都已支持通过 Observables 进行更新，但平移、相机等属性已经支持，并且可以与 Makie 的标准动画 API 结合使用。

```
<input id="hidecode2" class="hidecode" type="checkbox">
```

```julia
# Example inspiration and Lego model by https://github.com/Kevin-Mattheus-Moerman
# https://twitter.com/KMMoerman/status/1417759722963415041
using MeshIO, FileIO, GeometryBasics, RPRMakie

colors = Dict(
    "eyes" => "#000",
    "belt" => "#000059",
    "arm" => "#009925",
    "leg" => "#3369E8",
    "torso" => "#D50F25",
    "head" => "yellow",
    "hand" => "yellow"
)

origins = Dict(
    "arm_right" => Point3f(0.1427, -6.2127, 5.7342),
    "arm_left" => Point3f(0.1427, 6.2127, 5.7342),
    "leg_right" => Point3f(0, -1, -8.2),
    "leg_left" => Point3f(0, 1, -8.2),
)

rotation_axes = Dict(
    "arm_right" => Vec3f(0.0000, -0.9828, 0.1848),
    "arm_left" => Vec3f(0.0000, 0.9828, 0.1848),
    "leg_right" => Vec3f(0, -1, 0),
    "leg_left" => Vec3f(0, 1, 0),
)

function plot_part!(scene, parent, name::String)
    m = load(assetpath("lego_figure_" * name * ".stl"))
    color = colors[split(name, "_")[1]]
    trans = Transformation(parent)
    ptrans = Makie.transformation(parent)
    origin = get(origins, name, nothing)
    if !isnothing(origin)
        centered = m.position .- origin
        m = GeometryBasics.Mesh(meta(centered; normals=m.normals), faces(m))
        translate!(trans, origin)
    else
        translate!(trans, -ptrans.translation[])
    end
    return mesh!(scene, m; color=color, transformation=trans)
end

function plot_lego_figure(s, floor=true)
    # Plot hierarchical mesh!
    figure = Dict()
    # Plot hierarchical mesh!
    figure["torso"] = plot_part!(s, s, "torso")
        figure["head"] = plot_part!(s, figure["torso"], "head")
            figure["eyes_mouth"] = plot_part!(s, figure["head"], "eyes_mouth")
        figure["arm_right"] = plot_part!(s, figure["torso"], "arm_right")
            figure["hand_right"] = plot_part!(s, figure["arm_right"], "hand_right")
        figure["arm_left"] = plot_part!(s, figure["torso"], "arm_left")
            figure["hand_left"] = plot_part!(s, figure["arm_left"], "hand_left")
        figure["belt"] = plot_part!(s, figure["torso"], "belt")
            figure["leg_right"] = plot_part!(s, figure["belt"], "leg_right")
            figure["leg_left"] = plot_part!(s, figure["belt"], "leg_left")
    # lift the little guy up
    translate!(figure["torso"], 0, 0, 20)
    # add some floor
    floor && mesh!(s, Rect3f(Vec3f(-400, -400, -2), Vec3f(800, 800, 2)), color=:white)
    return figure
end

RPRMakie.activate!(iterations=200, plugin=RPR.Northstar)
radiance = 50000
lights = [
    EnvironmentLight(1.5, rotl90(load(assetpath("sunflowers_1k.hdr"))')),
    PointLight(Vec3f(50, 0, 200), RGBf(radiance, radiance, radiance*1.1)),
]
s = Scene(size=(500, 500), lights=lights)

cam3d!(s)
c = cameracontrols(s)
c.near[] = 5
c.far[] = 1000
update_cam!(s, c, Vec3f(100, 30, 80), Vec3f(0, 0, -10))
figure = plot_lego_figure(s)

rot_joints_by = 0.25*pi
total_translation = 50
animation_strides = 10

a1 = LinRange(0, rot_joints_by, animation_strides)
angles = [a1; reverse(a1[1:end-1]); -a1[2:end]; reverse(-a1[1:end-1]);]
nsteps = length(angles); #Number of animation steps
translations = LinRange(0, total_translation, nsteps)

Makie.record(s, "lego_walk.mp4", zip(translations, angles)) do (translation, angle)

    # Rotate right arm + hand
    for name in ["arm_left", "arm_right", "leg_left", "leg_right"]
        rotate!(figure[name], rotation_axes[name], angle)
    end
    translate!(figure["torso"], translation, 0, 20)
end
```

```
<label for="hidecode2" class="hidecode"></label>
```

```
<video mute autoplay controls src="/assets/lego_walk.mp4">
</video>
```

## 地球示例

```
<input id="hidecode1" class="hidecode" type="checkbox">
```

```julia
# by Lazaro Alonso
# taken from: https://lazarusa.github.io/BeautifulMakie/GeoPlots/submarineCables3D/
using GeoMakie, Downloads
using GeoJSON, GeoInterface
using FileIO
using RPRMakie
# data from
# https://github.com/telegeography/www.submarinecablemap.com
urlPoints = "https://raw.githubusercontent.com/telegeography/www.submarinecablemap.com/master/web/public/api/v3/landing-point/landing-point-geo.json"
urlCables = "https://raw.githubusercontent.com/telegeography/www.submarinecablemap.com/master/web/public/api/v3/cable/cable-geo.json"

landPoints = Downloads.download(urlPoints, IOBuffer())
landCables = Downloads.download(urlCables, IOBuffer())

land_geoPoints = GeoJSON.read(seekstart(landPoints))
land_geoCables = GeoJSON.read(seekstart(landCables))

toPoints = GeoMakie.geo2basic(land_geoPoints)
feat = GeoInterface.features(land_geoCables)
toLines = GeoInterface.coordinates.(GeoInterface.geometry.(feat))

# broken lines at -180 and 180... they should
# be the same line and be in the same array.

# some 3D transformations
function toCartesian(lon, lat; r = 1.02, cxyz = (0, 0, 0))
    x = cxyz[1] + r * cosd(lat) * cosd(lon)
    y = cxyz[2] + r * cosd(lat) * sind(lon)
    z = cxyz[3] + r * sind(lat)
    return (x, y, z)
end

toPoints3D = [Point3f([toCartesian(point[1], point[2])...]) for point in toPoints]

splitLines3D = []
for i in 1:length(toLines)
    for j in 1:length(toLines[i])
        ptsLines = toLines[i][j]
        tmp3D = []
        for k in 1:length(ptsLines)
            x, y = ptsLines[k]
            x, y, z = toCartesian(x, y)
            push!(tmp3D, [x, y, z])
        end
        push!(splitLines3D, Point3f.(tmp3D))
    end
end

earth_img = load(Downloads.download("https://upload.wikimedia.org/wikipedia/commons/5/56/Blue_Marble_Next_Generation_%2B_topography_%2B_bathymetry.jpg"))
# the actual plot !
RPRMakie.activate!(; iterations=100)
scene = with_theme(theme_dark()) do
    fig = Figure(; size=(1000, 1000))
    radiance = 30
    lights = [EnvironmentLight(0.5, load(RPR.assetpath("starmap_4k.tif"))),
              PointLight(Vec3f(1, 1, 3), RGBf(radiance, radiance, radiance))]
    ax = LScene(fig[1, 1]; show_axis=false, scenekw=(;lights=lights))
    n = 1024 ÷ 4 # 2048
    θ = LinRange(0, pi, n)
    φ = LinRange(-pi, pi, 2 * n)
    xe = [cos(φ) * sin(θ) for θ in θ, φ in φ]
    ye = [sin(φ) * sin(θ) for θ in θ, φ in φ]
    ze = [cos(θ) for θ in θ, φ in φ]
    surface!(ax, xe, ye, ze; color=earth_img)
    meshscatter!(toPoints3D; color=1:length(toPoints3D), markersize=0.005, colormap=:plasma)
    colors = Makie.default_palettes.color[]
    c = Iterators.cycle(colors)
    foreach(((l, c),) -> lines!(ax, l; linewidth=2, color=c), zip(splitLines3D, c))
    ax.scene.camera_controls.eyeposition[] = Vec3f(1.5)
    return ax.scene
end

save("submarin_cables.png", scene)
```

```
<label for="hidecode1" class="hidecode"></label>
```

```
<img src="/assets/earth.png">
```
