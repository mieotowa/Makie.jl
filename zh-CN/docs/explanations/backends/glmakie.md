# GLMakie

[GLMakie](https://github.com/MakieOrg/Makie.jl/tree/master/GLMakie) 是一个原生的、基于桌面的后端，也是功能最完备的。
它需要配备支持 OpenGL 版本 3.3 或更高版本的显卡。
It requires an OpenGL enabled graphics card with OpenGL version 3.3 or higher.

## 激活 GLMakie 与 screen 配置

通过调用 `GLMakie.activate!()` 函数来激活该后端：

```julia:docs
# hideall
using GLMakie, Markdown
println("~~~")
println(Markdown.html(@doc GLMakie.activate!))
println("~~~")
```

\textoutput{docs}

#### 窗口缩放

图形的尺寸以与显示器无关的“逻辑”单位给出，GLMakie 后端会在高分辨率（HiDPI）/Retina 显示器上自动调整显示窗口的大小。
例如，默认 `size = (800, 600)` 在配置了 200% 缩放因子的 HiDPI 显示器上将以 1600 × 1200 的窗口展示。
For example, the default `size = (800, 600)` will be shown in a 1600 × 1200 window
on a HiDPI display which is configured with a 200% scaling factor.

可以通过显示图形时指定不同的 `scalefactor` 值来覆盖缩放因子：

```julia
fig = Figure(size = (800, 600))
# ...
display(fig, scalefactor = 1.5)
```

如果不改变默认的自动配置，当在具有不同缩放因子的显示器之间移动窗口时，Windows 和 OSX 上的窗口会自动调整大小以保持其外观尺寸不变。
（X11 不支持独立的缩放因子，且当前底层的 GLFW 库未编译为支持 Wayland）
(Independent scaling factors are not supported by X11, and at this time the underlying
GLFW library is not compiled with Wayland support.)

#### 分辨率缩放

Related to the window scaling factor, the mapping from figure sizes and positions to pixels
can be scaled to achieve HiDPI/Retina resolution renderings.
与窗口缩放因子相关，从图形尺寸和位置到像素的映射可以进行缩放，以实现高分辨率（HiDPI）/Retina 渲染。
分辨率缩放默认与窗口缩放因子相同，但可通过显示图形时的 `px_per_unit` 参数独立覆盖：

```julia
fig = Figure(size = (800, 600))
# ...
display(fig, px_per_unit = 2)
```

保存 PNG 图像时也可以更改分辨率缩放因子：

```julia
save("hires.png", fig, px_per_unit = 2)    # 1600 × 1200 px png
save("lores.png", fig, px_per_unit = 0.5)  #  400 × 300 px png
```

如果脚本可能在原生屏幕 DPI 可变的交互环境中运行，保存图形时明确设置 `px_per_unit = 1` 可确保结果的一致性。

#### 多窗口

GLMakie has experimental support for displaying multiple independent figures (or scenes). GLMakie 支持实验性的多窗口显示独立图形（或场景）功能。 要打开新窗口，使用 `display(GLMakie.Screen(), figure_or_scene)`。 要关闭所有窗口，使用 `GLMakie.closeall()`。 To close all windows, use `GLMakie.closeall()`.

## 在 Linux 中强制使用独立 GPU

Normally the dedicated GPU is used for rendering.
通常情况下，渲染会使用独立 GPU。
如果系统使用集成 GPU，可以在启动 julia 时在 bash 终端中使用 `$ sudo DRI_PRIME=1 julia` 告诉 Julia 使用独立 GPU。
要永久使用独立 GPU，可以在 `.bashrc` 或 `.zshrc` 文件中添加一行 `export DRI_PRIME=1`。
To have it permanently used, add the line `export DRI_PRIME=1` in  your `.bashrc` or `.zshrc` file.

## OpenGL 故障排除

如果您在加载 GLMakie 时遇到任何错误，这很可能意味着您没有支持 OpenGL 的显卡，或者您没有安装支持 OpenGL 3.3 的驱动程序。
请注意，大多数 GPU，即使是8年前的集成 GPU，也支持 OpenGL 3.3。
Note that most GPUs, even 8 year old integrated ones, support OpenGL 3.3.

在 Linux 上，您可以使用以下命令查看您的 OpenGL 版本:
`glxinfo | grep "OpenGL version"`

如果您在 Linux 上使用 AMD 或 Intel GPU，您可能会遇到 [GLFW#198](https://github.com/JuliaGL/GLFW.jl/issues/198) 问题。

在无头服务器上，仍需安装 X 服务器和合适的图形驱动程序

您可以在这篇 [nextjournal 文章](https://nextjournal.com/sdanisch/GLMakie-nogpu) 中找到如何设置的示例。

GLMakie 的 CI 环境没有 GPU，所以您也可以查看 [.github/workflows/glmakie.yaml](https://github.com/MakieOrg/Makie.jl/blob/master/.github/workflows/glmakie.yaml) 了解一个可行的设置。

如果上述方法都不奏效，您可以查看其他 [后端](/explanations/backends/)，它们均无需 GPU 即可工作。

如果您遇到指向 [GLFW.jl](https://github.com/JuliaGL/GLFW.jl) 的错误，请查看现有的 [GLFW 问题](https://github.com/JuliaGL/GLFW.jl/issues)，同时在谷歌（必应等搜索引擎）上搜索这些错误。 这很可能是需要在 [glfw c库](https://github.com/glfw/glfw) 或 GPU 驱动程序中修复的问题。 This is then very likely something that needs fixing in the  [glfw c library](https://github.com/glfw/glfw) or in the GPU drivers.

!!! warning
GLMakie is not thread-safe! Makie functions to display in GLMakie or updates to `Observable` displayed in GLMakie windows from other threads may not work as expected or cause a segmentation fault.

## WSL 安装或 X 转发设置

来自：[Microsoft/WSL/issues/2855](https://github.com/Microsoft/WSL/issues/2855#issuecomment-358861903)

WSL 可以正常运行 OpenGL，但这并非官方支持的场景。
从商店安装干净的 Ubuntu 后执行以下操作：
From a clean Ubuntu install from the store do:

```
sudo apt install ubuntu-desktop mesa-utils
export DISPLAY=localhost:0
glxgears
```

在 Windows 端：

1. 安装 [VcXsrv](https://sourceforge.net/projects/vcxsrv/)
2. 选择“多个窗口”-> 显示器 0 -> 启动无客户端 -> 禁用原生 OpenGL

故障处理：

1.)  安装：`sudo apt-get install -y xorg-dev mesa-utils xvfb libgl1 freeglut3-dev libxrandr-dev libxinerama-dev libxcursor-dev libxi-dev libxext-dev`

2.) WSL 在传递 localhost 时存在一些问题，可能需要使用：`export DISPLAY=192.168.178.31:0`，其中 `192.168.178.31` 是运行 VcXsrv 的 PC 网络适配器的本地 IP 地址

3.) 可能需要执行 `mv /opt/julia-1.5.2/lib/julia/libstdc++.so.6 /opt/julia-1.5.2/lib/julia/libcpp.backup`，这是 [GLFW#198](https://github.com/JuliaGL/GLFW.jl/issues/198) 的另一种表现形式

## 在 macOS 上 GLMakie 无法显示图形或全屏模式下崩溃

当图形用户界面（GUI）不是从 AppBundle 启动时，macOS 会发出警告，并可能导致引发此警告的 Julia 进程崩溃。
只有当 macOS 设置->桌面与Dock->菜单栏->自动隐藏和显示菜单栏未设置为“永不”时，才会出现此警告。
因此，请确保将此设置设为“永不”，以允许在 macOS 上使用 GLMakie。
This warning only occurs if macOS Settings->Desktop & Dock->Menu Bar->Automatically hide and show the menu bar is not set to Never.
Therefore make sure this setting is set to `Never` to enable the use of GLMakie on macOS.
