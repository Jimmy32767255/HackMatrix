# HackMatrix Code Wiki

## 1. 项目概述

HackMatrix 是一个基于 C++ 的 3D Linux 桌面环境（也可作为游戏引擎使用）。该项目采用模块化架构，集成了 Wayland 窗口管理、3D 渲染、游戏引擎特性和多人游戏支持。

### 1.1 核心特性

- **3D 虚拟桌面**：在 3D 空间中运行和管理传统 X11/Wayland 应用窗口
- **窗口管理系统**：支持窗口焦点切换、缩放、移动等操作
- **游戏引擎**：基于 ECS（Entity Component System）架构的实体组件系统
- **3D 渲染引擎**：使用 OpenGL 进行实时渲染，支持阴影、纹理、光照等
- **多人游戏支持**：通过 ENet 实现网络同步
- **脚本系统**：支持 C++、JavaScript、Python 脚本扩展
- **持久化存储**：使用 SQLite 存储游戏状态和实体数据

### 1.2 技术栈

- **语言**：C++20, C11, Python 3
- **构建系统**：CMake
- **图形 API**：OpenGL, GLM 数学库
- **窗口系统**：Wayland (wlroots), X11
- **网络库**：ENet, ZeroMQ
- **持久化**：SQLite (SQLiteCpp)
- **序列化**：Protocol Buffers, Cereal
- **日志**：spdlog, fmt
- **UI**：Dear ImGui

---

## 2. 项目架构

### 2.1 顶层目录结构

```
/workspace/
├── include/           # 头文件目录
│   ├── components/     # ECS 组件
│   ├── systems/        # ECS 系统
│   ├── MultiPlayer/    # 多人游戏客户端/服务器
│   ├── WindowManager/  # 窗口管理
│   ├── Voxel/         # 体素空间
│   ├── SQLiteCpp/     # SQLite 封装
│   ├── wayland/       # Wayland 相关
│   ├── cereal/        # 序列化库
│   ├── glm/           # 数学库
│   ├── imgui/         # UI 库
│   └── enet/          # 网络库
├── src/               # 源代码目录
│   ├── components/    # 组件实现
│   ├── systems/       # 系统实现
│   ├── MultiPlayer/   # 多人游戏实现
│   ├── WindowManager/ # 窗口管理实现
│   ├── Voxel/         # 体素实现
│   ├── sqlite/        # SQLite 实现
│   ├── wayland/       # Wayland 实现
│   └── imgui/         # UI 实现
├── client_libs/       # 客户端库
│   ├── python/        # Python 客户端
│   └── js/            # JavaScript 客户端
├── protos/            # Protocol Buffer 定义
├── shaders/           # GLSL 着色器
├── vox/               # 3D 模型资源
├── scripts/           # Python 脚本示例
├── tests/             # 单元测试
└── CMakeLists.txt     # 构建配置
```

### 2.2 核心模块依赖关系

```
┌─────────────────────────────────────────────────────────────┐
│                        Engine                               │
│  (主控制器，协调所有模块)                                     │
└────────────────┬────────────────────────────────────────────┘
                 │
    ┌────────────┼────────────┬─────────────┬──────────────┐
    ▼            ▼            ▼             ▼              ▼
┌────────┐  ┌────────┐  ┌──────────┐  ┌─────────┐  ┌──────────┐
│Renderer│  │ World  │  │ Controls │  │ Camera  │  │ WindowMgr│
│ 渲染器  │  │ 世界   │  │ 控制器    │  │ 摄像机   │  │ 窗口管理  │
└────────┘  └────────┘  └──────────┘  └─────────┘  └──────────┘
    │            │            │             │              │
    └────────────┴────────────┴─────────────┴──────────────┘
                                │
                    ┌──────────┴──────────┐
                    ▼                     ▼
            ┌─────────────┐      ┌─────────────┐
            │EntityRegistry│      │   MultiPlayer│
            │  (ECS核心)   │      │ (多人游戏)   │
            └─────────────┘      └─────────────┘
```

---

## 3. 核心模块详解

### 3.1 Engine 模块

**文件位置**：`src/engine.cpp`, `include/engine.h`

Engine 是整个应用的主控制器，负责初始化和协调所有子系统。

#### 关键类

**Engine 类**

```cpp
class Engine {
  World* world;                    // 世界管理
  Api* api;                        // 外部 API
  Renderer* renderer;              // 渲染器
  Controls* controls;              // 输入控制器
  Camera* camera;                   // 摄像机
  WindowManager::WindowManagerPtr wm; // 窗口管理器
  std::shared_ptr<EntityRegistry> registry; // ECS 注册表
  std::shared_ptr<MultiPlayer::Client> client; // 网络客户端
  std::shared_ptr<MultiPlayer::Server> server; // 网络服务器
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `Engine(char** envp, EngineOptions)` | 构造函数，初始化所有子系统 |
| `wire()` | 连接各子系统之间的引用关系 |
| `frame()` | 主渲染循环的一帧 |
| `action(Action)` | 执行游戏动作 |
| `getRegistry()` | 获取 ECS 注册表 |
| `getCamera()` | 获取摄像机指针 |
| `getRenderer()` | 获取渲染器指针 |
| `registerClient()` | 注册网络客户端 |
| `registerServer()` | 注册网络服务器 |

#### EngineOptions 结构

```cpp
struct EngineOptions {
  bool enableGui = true;           // 启用 ImGui
  bool enableControls = true;       // 启用控制器
  bool invertYAxis = false;         // 反转 Y 轴
};
```

---

### 3.2 Renderer 模块

**文件位置**：`src/renderer.cpp`, `include/renderer.h`

Renderer 负责所有 OpenGL 渲染操作，包括 3D 场景、应用窗口、线条、体素等。

#### 关键类

**Renderer 类**

```cpp
class Renderer {
  // OpenGL 资源
  GlBuffer APP_VBO, DIRECT_RENDER_VBO, CURSOR_VBO, LINE_VBO;
  GlVertexArray APP_VAO, DIRECT_RENDER_VAO, CURSOR_VAO, LINE_VAO;
  GlVertexArray MESH_VERTEX, DYNAMIC_OBJECT_VERTEX, VOXEL_SELECTIONS;
  
  // 着色器
  Shader* shader;        // 主着色器
  Shader* cameraShader;  // 摄像机着色器
  Shader* appShader;     // 应用窗口着色器
  Shader* depthShader;   // 深度着色器
  Shader* cursorShader;  // 光标着色器
  
  // 渲染数据
  VoxelSpace voxelSpace;              // 体素空间
  RenderedVoxelSpace voxelMesh;       // 体素网格
  int verticesInMesh = 0;             // 网格顶点数
  int verticesInDynamicObjects = 0;   // 动态对象顶点数
  
  // 配置
  bool voxelsEnabled = true;           // 体素启用
  bool shadowsEnabled = true;          // 阴影启用
  bool isWireframe = false;           // 线框模式
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `render(perspective, fromLight)` | 执行主渲染 |
| `updateChunkMeshBuffers(meshes)` | 更新区块网格缓冲 |
| `updateDynamicObjects(obj)` | 更新动态对象 |
| `addLine(index, line)` | 添加线条 |
| `addVoxels(positions, size, color)` | 添加体素 |
| `clearVoxelsInBox(min, max)` | 清除区域内的体素 |
| `toggleWireframe()` | 切换线框模式 |
| `toggleMeshing()` | 切换网格化 |
| `screenshotFromCurrentFramebuffer()` | 截图 |
| `wireWindowManager(wm, space)` | 连接窗口管理器 |

#### 渲染视角

```cpp
enum RenderPerspective {
  CAMERA,  // 摄像机视角
  LIGHT    // 光源视角（用于阴影贴图）
};
```

---

### 3.3 World 模块

**文件位置**：`src/world.cpp`, `include/world.h`

World 负责管理 3D 世界中的所有元素，包括方块（Chunk）、动态对象、线条等。

#### 关键类

**World 类**

```cpp
class World : public WorldInterface {
  // 区块管理
  deque<deque<shared_ptr<Chunk>>> chunks;      // 世界中的区块
  map<DIRECTION, deque<future<deque<shared_ptr<Chunk>>>>> preloadedChunks;
  int WORLD_SIZE = 9;                         // 世界大小
  int PRELOAD_SIZE = 3;                       // 预加载大小
  
  // 世界元素
  vector<Line> lines;                          // 线条
  shared_ptr<DynamicObjectSpace> dynamicObjects; // 动态对象空间
  shared_ptr<DynamicCube> dynamicCube;         // 动态方块
  
  // 常量
  const float CUBE_SIZE = 0.1;                 // 方块大小
  Loader* loader;                              // 加载器
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `tick()` | 世界更新tick |
| `attachRenderer(renderer)` | 绑定渲染器 |
| `getLookedAtCube()` | 获取当前注视的方块 |
| `addCube(x, y, z, blockType)` | 添加方块 |
| `removeCube(position)` | 移除方块 |
| `addLine(line)` | 添加线条 |
| `save(filename)` | 保存世界 |
| `load(filename)` | 加载世界 |
| `mesh(realTime)` | 生成网格 |
| `loadRegion(coordinate)` | 加载区域 |
| `loadLatest()` | 加载最新存档 |

#### 世界接口 WorldInterface

```cpp
class WorldInterface {
  virtual void tick() = 0;
  virtual void attachRenderer(Renderer*) = 0;
  virtual Position getLookedAtCube() = 0;
  virtual void addCube(int, int, int, int) = 0;
  virtual void addLine(Line) = 0;
  virtual void removeLine(Line) = 0;
  virtual void action(Action) = 0;
  virtual vector<Line> getLines() = 0;
  virtual void save(string) = 0;
  virtual void load(string) = 0;
  virtual void loadRegion(Coordinate) = 0;
  virtual void initLoader(string, shared_ptr<TexturePack>) = 0;
  virtual void loadLatest() = 0;
  virtual void mesh(bool = true) = 0;
  virtual ChunkMesh meshSelectedCube(Position) = 0;
  virtual shared_ptr<Chunk> getChunk(int, int) = 0;
  virtual shared_ptr<DynamicObjectSpace> getDynamicObjects() = 0;
};
```

---

### 3.4 Camera 模块

**文件位置**：`src/camera.cpp`, `include/camera.h`

Camera 负责管理摄像机位置、朝向、视锥体等。

#### 关键类

**Camera 类**

```cpp
class Camera {
  // 位置和朝向
  glm::vec3 position;      // 摄像机位置
  glm::vec3 front;        // 朝向向量
  glm::vec3 up;           // 上向量
  
  // 旋转角度
  float yaw;              // 偏航角
  float pitch;            // 俯仰角
  
  // 矩阵
  glm::mat4 viewMatrix;           // 视图矩阵
  glm::mat4 projectionMatrix;    // 投影矩阵
  
  // 移动
  queue<Movement> movements;    // 移动队列
  float cameraSpeed;             // 移动速度
  static float DEFAULT_CAMERA_SPEED;
  
  // 投影参数
  float zFar, zNear, yFov;
};
```

#### Movement 结构

```cpp
struct Movement {
  glm::vec3 startPosition;
  glm::vec3 finishPosition;
  glm::vec3 startFront;
  glm::vec3 finishFront;
  double startTime;
  double endTime;
  std::shared_ptr<bool> isDone;
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `handleTranslateForce(...)` | 处理平移输入 |
| `handleRotateForce(window, dx, dy)` | 处理旋转输入 |
| `tick()` | 更新摄像机状态 |
| `moveTo(pos, front, seconds)` | 平滑移动到目标 |
| `isMoving()` | 检查是否在移动中 |
| `getViewMatrix()` | 获取视图矩阵 |
| `getProjectionMatrix()` | 获取投影矩阵 |
| `createFrustum()` | 创建视锥体 |
| `changeSpeed(delta)` | 改变移动速度 |

#### 视锥体 (Frustum)

```cpp
struct Frustum {
  Plane topFace;
  Plane bottomFace;
  Plane rightFace;
  Plane leftFace;
  Plane farFace;
  Plane nearFace;
};

struct Plane {
  glm::vec3 normal;
  float distance;
  float getSignedDistanceToPlane(const glm::vec3& point) const;
};
```

---

### 3.5 Controls 模块

**文件位置**：`src/controls.cpp`, `include/controls.h`

Controls 负责处理所有用户输入，包括键盘、鼠标事件。

#### 关键类

**Controls 类**

```cpp
class Controls {
  ControlMappings controlMappings;    // 按键映射
  WindowManager::WindowManagerPtr wm;   // 窗口管理器
  World* world;                         // 世界
  Camera* camera;                       // 摄像机
  Renderer* renderer;                  // 渲染器
  
  // 状态
  bool grabbedCursor = true;           // 是否锁定光标
  bool appFocused = false;             // 应用是否聚焦
  
  // 事件队列
  std::queue<KeysymEvent> keysymQueue; // 按键事件队列
  std::vector<DeferedAction> deferedActions; // 延迟动作
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `poll(window, camera, world)` | 轮询输入状态 |
| `mouseCallback(window, x, y)` | 鼠标回调 |
| `handleKeySym(sym, pressed, mod, shift)` | 处理按键事件 |
| `moveTo(pos, rotation, secs, callback)` | 平滑移动 |
| `goToApp(entity)` | 聚焦到应用 |
| `applyMovementInput(...)` | 应用移动输入 |
| `applyLookDelta(dx, dy)` | 应用视角移动 |
| `handlePointerButton(button, pressed)` | 处理指针按钮 |
| `disableKeys()` / `enableKeys()` | 启用/禁用键盘 |
| `triggerScreenshot()` | 触发截图 |
| `runQueuedActions()` | 执行队列中的动作 |

---

### 3.6 Chunk 模块

**文件位置**：`src/chunk.cpp`, `include/chunk.h`

Chunk 负责管理 3D 世界中的区块，包含方块数据。

#### 关键类

**Chunk 类**

```cpp
class Chunk {
  int posX, posY, posZ;                    // 区块位置
  vector<shared_ptr<Cube>> data;           // 方块数据
  shared_ptr<Mesher> mesher;               // 网格生成器
  
  // 缓存
  ChunkMesh cachedSimpleMesh;              // 简单网格缓存
  ChunkMesh cachedGreedyMesh;              //贪婪网格缓存
  bool damagedSimple = true;               // 简单网格是否损坏
  bool damagedGreedy = true;               // 贪婪网格是否损坏
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `getCube(x, y, z)` | 获取方块 |
| `addCube(cube, x, y, z)` | 添加方块 |
| `removeCube(x, y, z)` | 移除方块 |
| `mesh()` | 生成网格 |
| `meshAsync()` | 异步生成网格 |
| `meshedFaceFromPosition(position)` | 获取某位置的网格面 |

#### 区块坐标结构

```cpp
struct ChunkPosition {
  int x, y, z;
};

struct ChunkCoords {
  int x, y, z;
};
```

---

### 3.7 WindowManager 模块

**文件位置**：`src/WindowManager/`, `include/WindowManager/`

WindowManager 负责管理 X11/Wayland 应用窗口。

#### 关键类

**WindowManagerInterface (抽象基类)**

```cpp
class WindowManagerInterface {
  virtual ~WindowManagerInterface() = default;
  virtual void unfocusApp() = 0;
  virtual void menu() = 0;
  virtual void createAndRegisterApps(char**) = 0;
  virtual optional<entt::entity> getCurrentlyFocusedApp() = 0;
  virtual void focusApp(entt::entity) = 0;
  virtual void wire(shared_ptr<WindowManagerInterface>, Camera*, Renderer*) = 0;
  virtual std::shared_ptr<Space> getSpace() = 0;
  virtual void registerControls(Controls*) = 0;
  virtual entt::entity registerWaylandApp(...) = 0;
  virtual vector<optional<entt::entity>> getAppsWithHotKeys() = 0;
  virtual void swapHotKeys(int, int) = 0;
};
```

**WindowManager 实现类**

```cpp
class WindowManager : public WindowManagerInterface {
  shared_ptr<EntityRegistry> registry;
  Display* display = NULL;
  bool waylandMode = false;
  Controls* controls = NULL;
  
  // 窗口映射
  map<Window, entt::entity> dynamicApps;
  optional<entt::entity> currentlyFocusedApp;
  vector<optional<entt::entity>> appsWithHotKeys;
  
  // 线程
  std::thread substructureThread;
  std::atomic_bool screenshotRequested = false;
};
```

**Space 类**

Space 表示窗口的 3D 空间信息。

```cpp
class Space {
  glm::vec3 position;         // 位置
  glm::vec3 rotation;         // 旋转 (degrees)
  glm::vec3 scale;            // 缩放
  Bootable* bootable;         // 可启动组件
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `focusApp(entity)` | 聚焦窗口 |
| `unfocusApp()` | 取消聚焦 |
| `menu()` | 打开应用菜单 |
| `createAndRegisterApps(envp)` | 创建并注册应用 |
| `registerWaylandApp(app, ...)` | 注册 Wayland 应用 |
| `swapHotKeys(a, b)` | 交换热键 |
| `setCursorVisible(visible)` | 设置光标可见性 |

---

### 3.8 Entity System (ECS)

**文件位置**：`include/entity.h`, `src/entity.cpp`

项目使用 EnTT 库实现 Entity Component System。

#### 关键类

**EntityRegistry 类**

```cpp
class EntityRegistry : public entt::registry {
  std::shared_ptr<SQLite::Database> db;      // 数据库连接
  std::vector<std::shared_ptr<SQLPersister>> persisters; // 持久化器
  std::map<int, entt::entity> entityLocator;  // 实体定位器
};
```

#### 核心方法

| 方法名 | 说明 |
|--------|------|
| `createPersistent()` | 创建可持久化实体 |
| `save(entity)` | 保存实体 |
| `load(entity)` | 加载实体 |
| `saveAll()` | 保存所有实体 |
| `loadAll()` | 加载所有实体 |
| `depersist(entity)` | 移除持久化 |
| `locateEntity(id)` | 定位实体 |
| `getDatabase()` | 获取数据库引用 |

---

### 3.9 Components (组件)

组件定义实体的数据属性。

#### 3.9.1 Bootable 组件

**文件**：`include/components/Bootable.h`

管理窗口应用的启动和渲染。

```cpp
struct Bootable {
  static constexpr unsigned int DEFAULT_WIDTH = 400;
  static constexpr unsigned int DEFAULT_HEIGHT = 300;
  
  shared_ptr<AppSurface> app;         // 应用表面
  glm::vec3 position;                   // 位置
  glm::vec3 rotation;                  // 旋转
  glm::vec3 scale;                     // 缩放
  int hotkeyIndex = -1;                 // 热键索引
  bool accessory = false;               // 是否为辅助窗口
};
```

#### 3.9.2 Parent 组件

**文件**：`include/components/Parent.h`

定义实体的父子关系。

```cpp
struct Parent {
  std::vector<int> childrenIds;        // 子实体ID列表
};
```

#### 3.9.3 Light 组件

**文件**：`include/components/Light.h`

定义光源属性。

```cpp
class Light {
  glm::vec3 color;                      // 颜色
  unsigned int depthMapFBO;             // 深度贴图帧缓冲
  unsigned int depthCubemap;            // 深度立方体贴图
  unsigned int textureUnit;             // 纹理单元
  std::vector<glm::mat4> shadowTransforms; // 阴影变换矩阵
  float nearPlane, farPlane;            // 裁剪平面
};
```

#### 3.9.4 Key 组件

**文件**：`include/components/Key.h`

定义钥匙和锁的交互逻辑。

```cpp
struct Key {
  int lockable;                         // 可锁定的实体ID
  TurnState state;                       // 状态 (TURNED/TURNING/UNTURNED/UNTURNING)
  RotateMovement turnMovement;          // 转动动画
  RotateMovement unturnMovement;        // 反转动动画
};
```

#### 3.9.5 Scriptable 组件

**文件**：`include/components/Scriptable.h`

支持脚本化实体。

```cpp
class Scriptable {
  std::string script;                    // 脚本内容
  ScriptLanguage language;               // 语言类型 (CPP/JAVASCRIPT/PYTHON)
  bool isDamaged;                        // 是否损坏需更新
  std::mutex _mutex;                     // 线程安全
};
```

#### 3.9.6 Player 组件

**文件**：`include/components/Player.h`

```cpp
class Player {
  uint32_t connectionId;                  // 连接ID
};
```

---

### 3.10 Systems (系统)

系统包含处理实体的逻辑。

#### 3.10.1 Update 系统

**文件**：`include/systems/Update.h`, `src/systems/Update.cpp`

```cpp
namespace systems {
  void updateAll(std::shared_ptr<EntityRegistry>, Renderer*);
  void update(std::shared_ptr<EntityRegistry>, entt::entity);
}
```

#### 3.10.2 Player 系统

**文件**：`include/systems/Player.h`, `src/systems/Player.cpp`

管理网络玩家的移动。

```cpp
namespace systems {
  void registerPlayer(std::shared_ptr<EntityRegistry>, uint32_t);
  void movePlayer(std::shared_ptr<EntityRegistry>, uint32_t, 
                  glm::vec3 position, glm::vec3 front, float time);
}
```

#### 3.10.3 Move 系统

**文件**：`include/systems/Move.h`, `src/systems/Move.cpp`

处理实体移动。

```cpp
namespace systems {
  void move(std::shared_ptr<EntityRegistry>);
}
```

#### 3.10.4 Door 系统

**文件**：`include/systems/Door.h`, `src/systems/Door.cpp`

处理门的开关逻辑。

```cpp
namespace systems {
  void door(std::shared_ptr<EntityRegistry>, World* world);
}
```

#### 3.10.5 KeyAndLock 系统

**文件**：`include/systems/KeyAndLock.h`, `src/systems/KeyAndLock.cpp`

处理钥匙和锁的交互。

```cpp
namespace systems {
  void keyAndLock(std::shared_ptr<EntityRegistry>);
}
```

#### 3.10.6 Light 系统

**文件**：`include/systems/Light.h`, `src/systems/Light.cpp`

处理光照渲染。

```cpp
namespace systems {
  void light(std::shared_ptr<EntityRegistry>, 
             std::function<void(glm::vec3, std::function<void()>)> renderDepthMap);
}
```

#### 3.10.7 Scripts 系统

**文件**：`include/systems/Scripts.h`, `src/systems/Scripts.cpp`

执行实体脚本。

```cpp
namespace systems {
  void scripts(std::shared_ptr<EntityRegistry>);
}
```

#### 3.10.8 ApplyTranslation 系统

**文件**：`include/systems/ApplyTranslation.h`, `src/systems/ApplyTranslation.cpp`

应用平移变换。

#### 3.10.9 ApplyRotation 系统

**文件**：`include/systems/ApplyRotation.h`, `src/systems/ApplyRotation.cpp`

应用旋转变换。

#### 3.10.10 Boot 系统

**文件**：`include/systems/Boot.h`, `src/systems/Boot.cpp`

处理实体启动逻辑。

#### 3.10.11 Intersections 系统

**文件**：`include/systems/Intersections.h`, `src/systems/Intersections.cpp`

处理碰撞检测。

#### 3.10.12 Derivative 系统

**文件**：`include/systems/Derivative.h`, `src/systems/Derivative.cpp`

计算导数/插值。

---

### 3.11 MultiPlayer 模块

**文件位置**：`src/MultiPlayer/`, `include/MultiPlayer/`

支持多人游戏功能。

#### 3.11.1 Client

**文件**：`include/MultiPlayer/Client.h`, `src/MultiPlayer/Client.cpp`

网络客户端。

```cpp
class Client {
  ENetHost* client;                      // ENet 主机
  ENetPeer* peer;                        // ENet 对等体
  bool _isConnected;                    // 连接状态
  double lastUpdate;                     // 上次更新时间
  double UPDATE_EVERY = 1.0 / 20.0;      // 更新频率 (20Hz)
  std::shared_ptr<EntityRegistry> registry;
};
```

#### 3.11.2 Server

**文件**：`include/MultiPlayer/Server.h`, `src/MultiPlayer/Server.cpp`

游戏服务器。

```cpp
class Server {
  ENetHost* server;                     // ENet 服务器
  std::atomic<bool> isRunning;          // 运行状态
  std::thread pollThread;               // 轮询线程
};
```

#### PlayerUpdate 结构

```cpp
struct PlayerUpdate {
  uint32_t playerID;                    // 玩家ID
  glm::vec3 position;                   // 位置
  glm::vec3 front;                      // 朝向
};
```

---

### 3.12 AppSurface 模块

**文件位置**：`include/AppSurface.h`, `src/app.cpp`

定义窗口表面的抽象接口。

#### 关键类

**AppSurface (抽象基类)**

```cpp
class AppSurface {
public:
  virtual ~AppSurface() = default;
  virtual void appTexture() = 0;
  virtual int createTexture() = 0;
  virtual void focus(unsigned long) = 0;
  virtual void unfocus(unsigned long) = 0;
  virtual void takeInputFocus() = 0;
  virtual void resize(int, int) = 0;
  virtual void resizeMove(int, int, int, int) = 0;
  virtual bool isFocused() = 0;
  virtual int getPID() = 0;
  virtual std::string getWindowName() = 0;
  virtual array<int, 2> getPosition() const = 0;
  virtual int getWidth() const = 0;
  virtual int getHeight() const = 0;
  virtual int getTextureId() const = 0;
  virtual glm::mat4 getHeightScalar() const = 0;
  virtual void select() = 0;
  virtual void deselect() = 0;
  virtual bool isSelected() = 0;
};
```

#### X11App 类

```cpp
class X11App : public AppSurface {
  Display* display;                      // X11 显示
  Window appWindow;                      // X11 窗口
  GLXFBConfig* fbConfigs;               // GLX 配置
  int textureUnit = -1;
  int textureId = -1;
  atomic_bool focused = false;
  atomic_bool selected = false;
  
  static X11App* byName(string, Display*, int, int, int);
  static X11App* byClass(string, Display*, int, int, int);
  static X11App* byWindow(Window, Display*, int, int, int);
  static X11App* byPID(int, Display*, int, int, int);
};
```

#### WaylandApp 类

定义在 `include/wayland_app.h`，Wayland 应用的实现。

---

## 4. 关键数据结构

### 4.1 Action 枚举

定义游戏中的各种动作。

```cpp
enum class Action {
  NONE,
  UP,
  DOWN,
  LEFT,
  RIGHT,
  Z_PLUS,
  Z_MINUS,
  INTERACT,
  DESTROY,
  SELECT,
  DEBUG,
  MENU,
  TOGGLE_WINDOW,
  TOGGLE_CURSOR,
  TOGGLE_WIREFRAME,
  TOGGLE_MESHING,
  SCREENSHOT,
  SAVE,
  CODE_BLOCK,
  PLAYER_SPEED_UP,
  PLAYER_SPEED_DOWN,
  WINDOW_FLOP,
  MAKE_WINDOW_BOOTABLE,
};
```

### 4.2 Line 结构

表示 3D 空间中的线条。

```cpp
struct Line {
  glm::vec3 start;                       // 起点
  glm::vec3 end;                         // 终点
  glm::vec3 color;                       // 颜色
  float size;                            // 粗细
};
```

### 4.3 Position 结构

世界坐标位置。

```cpp
struct Position {
  float x, y, z;
};
```

### 4.4 Cube 类

表示方块的基本类。

```cpp
class Cube {
  glm::vec3 position;                    // 位置
  int blockType;                         // 方块类型
  bool visible;                          // 是否可见
};
```

### 4.5 ChunkMesh 结构

存储区块网格数据。

```cpp
struct ChunkMesh {
  vector<float> vertices;               // 顶点数据
  vector<float> texCoords;              // 纹理坐标
  vector<int> blockTypes;               // 方块类型
  vector<float> selects;                 // 选择状态
};
```

### 4.6 DynamicObjectSpace

管理动态对象的空间划分。

```cpp
class DynamicObjectSpace {
  // 用于碰撞检测和查询的空间划分结构
};
```

---

## 5. 构建系统

### 5.1 CMake 配置

**文件**：`CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(Matrix LANGUAGES C CXX)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 20)

# 依赖管理
find_package(Crow REQUIRED)      # HTTP 服务器
find_package(Git QUIET)
pkg_check_modules(WLROOTS REQUIRED wlroots-0.19 wayland-server xkbcommon)
pkg_check_modules(GLFW REQUIRED glfw3)
pkg_check_modules(PROTOBUF REQUIRED protobuf)
```

### 5.2 编译目标

| 目标 | 说明 |
|------|------|
| `matrix` | 主程序 |
| `bootServer` | 启动服务器 |

### 5.3 依赖项

#### 系统依赖

- wayland-protocols
- wofi
- ZeroMQ (libzmq)
- Protocol Buffers (libprotobuf)
- spdlog (libspdlog)
- fmt (libfmt)
- GLFW (libglfw)
- OpenGL (libGL)
- pthread (libpthread)
- Assimp (libassimp)
- SQLite3 (libsqlite3)
- Crow (HTTP 服务器)

#### X11 依赖 (兼容性)

- X11 (libX11)
- Xcomposite (libXcomposite)
- Xtst (libXtst)
- Xext (libXext)
- Xfixes (libXfixes)

### 5.4 构建步骤

```bash
mkdir -p build
cd build
cmake ..
make -j$(nproc)
```

### 5.5 Protobuf 编译

Protocol Buffer 定义在 `protos/api.proto`，自动生成 C++ 和 Python 代码。

```bash
# 自动通过 CMake 触发
protoc --cpp_out=build/generated --python_out=client_libs/python protos/api.proto
```

---

## 6. 运行方式

### 6.1 主程序

```bash
./launch  # 从项目根目录启动
```

### 6.2 启动服务器 (多人游戏)

```bash
./build/tools/deployTools/bootServer
```

---

## 7. 客户端库

### 7.1 Python 客户端

**文件位置**：`client_libs/python/hackMatrix/`

```bash
# 安装
python -m venv hackmatrix_python
source hackmatrix_python/bin/activate
cd client_libs/python
pip install .

# 使用示例
python scripts/player-move.py
```

#### 核心 API

```python
import hackMatrix as hm

# 连接到服务器
client = hm.Client("localhost", 12345)

# 发送玩家位置更新
client.send_player(position, front)

# 断开连接
client.disconnect()
```

### 7.2 JavaScript 客户端

**文件位置**：`client_libs/js/`

```javascript
import { HackMatrix } from './client_libs/js/api.js';

const client = new HackMatrix('ws://localhost:12345');
```

---

## 8. 核心文件速查表

| 模块 | 头文件 | 源文件 |
|------|--------|--------|
| 引擎 | `include/engine.h` | `src/engine.cpp` |
| 渲染 | `include/renderer.h` | `src/renderer.cpp` |
| 世界 | `include/world.h` | `src/world.cpp` |
| 摄像机 | `include/camera.h` | `src/camera.cpp` |
| 控制 | `include/controls.h` | `src/controls.cpp` |
| 区块 | `include/chunk.h` | `src/chunk.cpp` |
| 窗口管理 | `include/WindowManager/WindowManager.h` | `src/WindowManager/WindowManager.cpp` |
| X11 应用 | `include/app.h` | `src/app.cpp` |
| 应用表面 | `include/AppSurface.h` | - |
| 实体系统 | `include/entity.h` | `src/entity.cpp` |
| 多人客户端 | `include/MultiPlayer/Client.h` | `src/MultiPlayer/Client.cpp` |
| 多人服务器 | `include/MultiPlayer/Server.h` | `src/MultiPlayer/Server.cpp` |
| 配置 | `include/Config.h` | `src/Config.cpp` |
| 持久化 | `include/persister.h` | `src/persister.cpp` |

---

## 9. 编码规范

### 9.1 命名约定

- **类名**：PascalCase (如 `EntityRegistry`, `WindowManager`)
- **方法名**：camelCase (如 `getCamera`, `handleRotateForce`)
- **成员变量**：下划线前缀或 camelCase (如 `world`, `cameraSpeed`)
- **枚举值**：大写下划线分隔 (如 `TURNED`, `PLAYER_UPDATE`)
- **常量**：大写下划线分隔 (如 `DEFAULT_CAMERA_SPEED`)

### 9.2 头文件结构

```cpp
#ifndef __MODULE_NAME_H__
#define __MODULE_NAME_H__

// 内容

#endif
```

### 9.3 代码格式化

项目使用 `.clang-format` 配置文件进行代码格式化。

---

## 10. 常见操作

### 10.1 截图

在 HackMatrix 中按 `p` 键保存截图到 `screenshots` 文件夹。

### 10.2 移动和视角

| 按键 | 功能 |
|------|------|
| 鼠标移动 | 旋转视角 |
| W/A/S/D | 前/左/后/右移动 |
| Q/E | 下降/上升 |
| Shift | 加速移动 |

### 10.3 窗口操作

| 按键 | 功能 |
|------|------|
| V | 打开应用菜单 |
| R | 聚焦窗口 |
| Super+E | 退出窗口模式 |
| Super+1-9 | 切换应用 |
| Super+Q | 关闭窗口 |

### 10.4 游戏引擎操作

| 按键 | 功能 |
|------|------|
| F | 进入鼠标模式 |
| ESC | 退出 HackMatrix |

---

## 11. 注意事项

### 11.1 已知问题

- 触摸板在移动时可能被禁用
- 滚动在窗口管理器模式下可能产生未定义状态
- Arch Linux 上的 Protobuf 编译错误（需使用 PR #55）
- X11 代码遗留（正在向纯 Wayland 迁移）

### 11.2 Wayland 支持

项目正在从 X11 向 Wayland 迁移。当前版本：
- 使用 wlroots 0.19
- 支持 Wayland 协议的原生窗口管理
- X11 代码作为兼容性保留

### 11.3 Crow 依赖

Crow HTTP 库优先通过 `find_package(Crow)` 查找，如未找到则从源码构建（git tag: `c61a26e`）。

```bash
# 强制从源码构建
cmake -S . -B build -DMATRIX_FORCE_FETCH_CROW=ON

# 使用本地 Crow 源码
cmake -S . -B build -DMATRIX_CROW_SOURCE_DIR=/path/to/Crow
```

---

## 12. 扩展指南

### 12.1 添加新组件

1. 在 `include/components/` 创建头文件
2. 在 `src/components/` 实现
3. 在 `EntityRegistry` 中注册持久化器
4. 实现对应的 SQLPersister

### 12.2 添加新系统

1. 在 `include/systems/` 创建头文件
2. 在 `src/systems/` 实现
3. 在 `Engine::frame()` 或 `World::tick()` 中调用

### 12.3 添加新着色器

1. 在 `shaders/` 创建 `.frag` 和 `.vert` 文件
2. 在 `Renderer` 中加载和使用

### 12.4 添加新 Protocol Buffer 消息

1. 编辑 `protos/api.proto`
2. 重新编译（CMake 会自动处理）
3. 在 C++ 和 Python 中使用生成的代码

---

本文档由代码分析自动生成，如有问题请联系项目维护者。
