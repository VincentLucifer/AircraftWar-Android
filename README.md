# Aircraft War · Android 飞机大战

[简体中文](README.md) | [English](README_EN.md)

使用 **Java 和 Android 原生组件**构建的纵向射击游戏，结合触摸操控、敌机与 Boss、难度变化、火力补给、本地排行榜和联机比分客户端，实践面向对象设计与 Android 游戏开发。

## 游戏内容

- **触摸控制与自动射击**：拖动英雄机躲避敌机和子弹。
- **多类敌机**：普通敌机、精英敌机与 Boss，分别组织移动、射击和补给逻辑。
- **三档难度**：Easy、Medium、Hard 使用独立游戏子类。
- **动态难度**：困难模式随时间调整敌机概率、生成频率、属性和 Boss 血量倍率。
- **补给系统**：包含生命补给、限时火力增强等道具；火力增强通过后台任务在计时后恢复。
- **音频反馈**：背景/Boss 音乐、命中、拾取和游戏结束音效，可在主页选择音乐开关。
- **成绩记录**：保存玩家名、分数、时间和排名，在应用私有文件中读写并按分数排序。
- **联机客户端**：通过 TCP Socket 进行匹配消息、比分交换和结算消息交互。

## 面向对象设计

| 设计方式 | 代码体现 | 目的 |
| --- | --- | --- |
| 单例 | `HeroAircraft` 的 volatile + 双重检查锁 | 统一当前英雄机实例 |
| 工厂 | `EnemyFactory` / `SupplyFactory` 及其实现 | 分离游戏循环与对象创建 |
| 策略 | `ShootStrategy`、`DirectShoot`、`DisperseShoot` | 将射击方式与飞机对象分离 |
| 模板化游戏流程 | `BaseGame` + `EasyGame/MediumGame/HardGame` | 复用更新、碰撞和绘制流程，调整难度规则 |
| DAO | `PlayerScoreDAO` / `PlayerScoreDAOImpl` | 分离成绩数据操作与界面 |

## 游戏循环与线程

`BaseGame` 继承 `SurfaceView`，通过 `SurfaceHolder.Callback` 管理绘制入口，后台线程执行游戏循环，Canvas 绘制背景、飞机、子弹、生命和分数。

单次更新包含敌机生成、射击、对象移动、碰撞检查和失效对象清理。游戏消息通过 `Handler` 传递到界面线程；网络接收线程也通过消息驱动比分显示。

## 项目结构

| 路径（`app/src/main/` 下） | 内容 |
| --- | --- |
| `java/.../activity/` | 主菜单、难度选择、游戏页面和成绩界面 |
| `java/.../game/` | SurfaceView 游戏循环与难度实现 |
| `java/.../aircraft/` / `bullet/` | 飞机与子弹对象 |
| `java/.../factory/` | 敌机与道具工厂 |
| `java/.../shoot/` | 射击策略 |
| `java/.../supply/` | 补给效果 |
| `java/.../DAO/` | 成绩模型与文件持久化 |
| `java/.../music/` | MediaPlayer / SoundPool 音频封装 |
| `res/` | 布局、图片与音效资源 |

## 构建与启动

| 配置 | 仓库设置 |
| --- | --- |
| Android Gradle Plugin | 8.0.2 |
| Gradle Wrapper | 8.0 |
| compileSdk / targetSdk | 33 / 33 |
| minSdk | 30（Android 11） |
| Java 源码兼容性 | Java 8 |

```bash
git clone https://github.com/VincentLucifer/AircraftWar-Android.git
cd AircraftWar-Android

# Linux / macOS
./gradlew assembleDebug

# Windows
gradlew.bat assembleDebug
```

也可使用 Android Studio 打开根目录，准备 Android SDK 33，完成 Gradle 同步后运行 `app`，设备或模拟器需满足 API 30 及以上。构建工具使用与 AGP 8 兼容的 JDK。

主页可选择音乐设置和游戏模式，离线模式进入难度选择，再启动游戏。

## 联机消息约定

客户端位于 `MainActivity.NetConn`，当前连接配置为 `10.0.2.2:9999`。联机模式需配合实现相同文本协议的服务端；`10.0.2.2` 用于 Android 模拟器访问宿主机，真机使用时应配置可达的服务器地址。

| 方向 | 消息示例 |
| --- | --- |
| 客户端 → 服务端 | `Match`、`Score:<score>`、`end:<score>` |
| 服务端 → 客户端 | `connecting`、`StartGame`、`opponentPlayer:<score>`、`EndGame` |

该部分展示 Socket、后台消息接收与 Android UI 更新之间的协作。

## 项目背景

基于 HITSZ 面向对象软件构造课程飞机大战项目进行 Android 实践，保留课程基础对象与资源，围绕游戏流程、设计模式、数据持久化和客户端交互展开。
