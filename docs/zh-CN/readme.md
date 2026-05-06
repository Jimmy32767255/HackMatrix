# HackMatrix


<img src="../../images/header_img.png" width="800">

一个3D Linux桌面环境（也可以作为游戏引擎）

<a href="https://www.youtube.com/watch?v=L6xDqNhGeEM">观看演示</a>

[加入Discord](https://discord.gg/Kx2rbJ8JCM)

[English](../../readme.md) | 简体中文

## 使用方法

### 在3D空间中导航
用鼠标环顾四周

<img src="../../vids/lookAround.gif" width="420">

使用以下按键移动 <br>
<img src="../../images/wasd.webp" width="100">

<img src="../../vids/move.gif" width="420">

### 打开窗口

HackMatrix 使用 `wofi`。

`d` 键被映射为移动，所以不按修饰键直接按 `v`

输入程序名称并按 `<enter>`

窗口将在你注视的位置打开。

按 `r` 键[聚焦](#手动聚焦窗口)窗口

<img src="../../vids/openWindow.gif" width="420">

### 手动聚焦窗口
注视窗口并按 `r`

__警告__: 有时由于bug，这会暂时失效。

如果发生这种情况，只需使用 Super+1 聚焦一个窗口，然后手动聚焦就会再次生效。


<img src="../../vids/focus.gif" width="420">

### 退出窗口
当窗口聚焦时按 `Super+e`

<img src="../../vids/exitWindow.gif" width="420">

### 窗口热键

窗口按照创建顺序自动分配热键。

使用 `Super+<数字>` 进行导航

<img src="../../vids/hotkey.gif" width="420">

### 关闭窗口

当窗口有焦点时，按 `Super+q`

### 退出 HackMatrix

当没有窗口聚焦时按 `<esc>`

（如果聚焦在窗口上，先按 `Super+e`）

### 截图

按 `p` 键将截图保存到 `<项目目录>/screenshots` 文件夹

### HackMatrix 菜单

HackMatrix 顶部有一个小菜单。

你可以用它来检查和修改实体（HackMatrix 的游戏引擎功能）

当没有聚焦应用时，按 `f` 进入鼠标模式

点击菜单左侧的箭头

导航到实体编辑器

有关游戏引擎和如何使用编辑器的更多信息，请参阅 [wiki页面](https://github.com/collinalexbell/HackMatrix/wiki/Game-Engine)。

## 编译/安装

### 依赖项

- wayland-protocols (`wayland-protocols`)（同时会引入 wayland 核心）
- wofi (`wofi`)
- ZeroMQ (`libzmq`)
- Protocol Buffers (`libprotobuf`)
- spdlog (`libspdlog`)
- fmt (`libfmt`)
- GLFW (`libglfw`)
- OpenGL (`libGL`)
- pthread (`libpthread`)
- Assimp (`libassimp`)
- SQLite3 (`libsqlite3`)
- Protobuf (`protobuf1`)
- 基础开发工具 (`basedevel`)

这个项目仍然不幸地保留了一些 X11 残留，这是由于一次半失败的 AI 迁移造成的，那次迁移本意是在添加 Wayland 支持的同时保留 X11，而不是替换 X11（这是我在开始 Wayland 工作后最终决定要做的事情）。很快会清理。
- X11 (`libX11`)
- Xcomposite (`libXcomposite`)
- Xtst (`libXtst`)
- Xext (`libXext`)
- Xfixes (`libXfixes`)
- XWinInfo (`x11-utils`)
- xdotool (`xdotool`)


你可以使用发行版的包管理器来安装这些库。以下是一些常见发行版的命令：

#### Ubuntu 或 Debian

```bash
sudo apt-get install wofi wayland-protocols rofi xdotool x11-utils protobuf-compiler build-essential libzmq3-dev libx11-dev libxcomposite-dev libxtst-dev libxext-dev libxfixes-dev libprotobuf-dev libspdlog-dev libfmt-dev libglfw3-dev libgl-dev libassimp-dev libsqlite3-dev pkgconf
```

#### Fedora 或 CentOS

```bash
sudo dnf install wayland-protocols wayland wofi rofi xdotool xorg-x11-utils protobuf-compiler @development-tools zeromq-devel libX11-devel libXcomposite-devel libXtst-devel libXext-devel libXfixes-devel protobuf-devel spdlog-devel fmt-devel glfw-devel mesa-libGL-devel assimp-devel sqlite-devel
```

#### Arch Linux

我目前正在处理 Arch 上的 protobuf 编译错误问题。[这个PR](https://github.com/collinalexbell/HackMatrix/pull/48)展示了如何解决这个问题。
如果你在 Arch 上并愿意帮助测试一个可以合并到 master 的 PR，请尝试[这个PR](https://github.com/collinalexbell/HackMatrix/pull/55)并在 PR 评论中告诉我它是否对你有效。非常感谢！

```bash
sudo pacman -S --needed wofi wayland-protocols xdotool rofi xorg-server xorg-xinit xorg-xwininfo xorg-xrandr protobuf base-devel zeromq libx11 libxcomposite libxtst libxext libxfixes spdlog fmt glfw-x11 mesa assimp sqlite
```

#### Gentoo
```bash
 sudo emerge --autounmask-write gui-apps/wofi dev-libs/wayland-protocols x11-misc/rofi net-libs/zeromq x11-libs/libX11 x11-libs/libXcomposite x11-libs/libXtst x11-libs/libXext x11-libs/libXfixes dev-libs/protobuf dev-libs/spdlog dev-libs/libfmt media-libs/glfw x11-libs/libGLw  dev-db/sqlite x11-misc/xdotool dev-libs/pthreadpool media-libs/assimp dmenu
```
> [!NOTE]
> 你可能会遇到 use flags 或 masked packages 的问题。你需要在自己的系统上解决这些问题。

## 构建

***警告*** 这是一个新的 Wayland 构建。它会有bug，但我还是合并到了 main 分支


```bash
mkdir -p build
cd build
cmake ..
make -j
```

## 运行

运行 `./launch`

### 如何让 client_libraries 工作
从 HackMatrix 根目录运行 `scripts/install-python-clientlib.sh` 进行自动安装

有关它运行的内容，请参阅下面的脚本

#### Python
```bash
# 安装 python hackMatrix 库
python -m venv hackmatrix_python
cd hackmatrix_python
source bin/activate
cd client_libs/python
pip install .
cd ../..

# 测试
python scripts/player-move.py
```


# 问题

## 打字时触摸板被禁用（移动时）
```
xinput list | grep -i touchpad
# 获取 id 并在下面的 <id> 中替换它
xinput set-prop <id> "libinput Disable While Typing Enabled" 0
```

# 想法
- 建模真实空间，比如我所在城市的公园，将它们放入 HackMatrix，然后改进设计并通过民主投票（用钱）决定应该实施哪个设计，然后在物理空间中执行设计

- 一种 HackMatrix 加密货币，用于资助各种 HackMatrix 项目

# Bugs
- 在 WM 模式下（未聚焦工作区时）滚动由未聚焦的窗口处理，导致只能通过聚焦和取消聚焦窗口来恢复的未定义状态。
