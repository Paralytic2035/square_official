# square_official

课程**官方 `square.cpp`** 的独立 ROS 包版本，用于和自写版做行为对照。

```
源码：d59ddbd34681f496aa6ba2615aa1b3c62bc4895b3a71c03d21de9d209f5bbbc6（244 行）
来源：hus-king/template2026_ws main 分支 src/mission/src/square.cpp
```

## 它和自写版的区别

| | 官方（本包） | 自写版 |
|---|---|---|
| 航点数量 | **固定 4 个**（`waypoints[4]` 数组） | 按 yaml 读，配几个飞几个 |
| yaml 配 5 个点 | **只飞前 4 个** | 5 个全飞 |
| 落点 | `(0,1)` 附近，**不回原点** | 飞回 `(0,0)` |
| 日志 | 每 50 ms 刷「到达目标点」 | 「飞向/到达航点 N」 |
| 悬停 | 无 | 起飞后悬停 2 s |

## 目录

```
square_official/
├── package.xml
├── CMakeLists.txt                  add_executable(square_official_node src/square.cpp)
├── launch/square.launch            支持 param_file 覆盖
├── config/
│   ├── square.yaml                 4 点 / 0.5 m / if_debug 0
│   └── square_hardware.yaml        实机（4 点 / 1.0 m / if_debug 0）
└── src/square.cpp                  官方源码，未修改
```

## 依赖

同自写版：`roscpp`、`mavros_msgs`、**`template`**（课程工作空间里的基础库，本包不含）。

## 安装与编译

```bash
cd ~/template2026_ws/src
git clone <本仓库地址>
cd ~/template2026_ws && catkin_make -j2
# 期望：traversing 3 packages / Built target square_official_node
```

> 可执行文件名是 `square_official_node`，避免和 `mission` 包里的 `square` 目标重名
> （catkin 的 CMake 目标名必须全局唯一）。

## 运行

```bash
# 仿真
roslaunch square_official square.launch

# 实机（必须带 param_file，否则读仿真默认 if_debug 1）
roslaunch square_official square.launch \
  param_file:=$HOME/template2026_ws/src/square_official/config/square_hardware.yaml
```

## 重要提醒

1. **它只飞 4 个点**。yaml 里写第 5 个点也不会被读——这是官方实现的限制，不是 bug。
2. **两份 yaml 的 `if_debug` 都是 0**（实机安全）。如果你要在**仿真**里跑，
   把它改成 1，否则程序会一直等 `Waiting for OFFBOARD mode` 不起飞。
3. 本包**不负责起飞控和定位**（那属于 `fly_demo`），只起任务节点。

## 为什么单独做个包

- 想对比“官方实现 vs 我的实现”的行为差异，需要一个能独立编译的官方版本
- 直接覆盖 `mission/src/square.cpp` 也行，但那样会反复覆盖共享代码；
  做成独立包后两份实现可以并存
