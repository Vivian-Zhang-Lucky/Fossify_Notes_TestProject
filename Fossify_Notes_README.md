# Fossify Notes Android 移动端测试项目

## 项目简介

本项目基于开源 Android 笔记应用 **Fossify Notes 1.7.0** 开展移动端软件测试实践，重点覆盖功能测试、移动端专项测试、ADB 命令行验证以及基础性能与稳定性测试。

测试基于 Android Emulator 完成，主要验证 Text Note、Checklist、搜索、Note 管理、PIN/Pattern 锁定、文件导入导出、数据持久化及典型 Android 场景下的应用表现，并输出完整测试文档与测试报告。

## 测试环境

| 项目 | 信息 |
|---|---|
| 测试对象 | Fossify Notes |
| 应用版本 | 1.7.0 |
| versionCode | 13 |
| 包名 | `org.fossify.notes` |
| 测试平台 | Android Emulator |
| 模拟设备 | `sdk_gphone64_arm64` |
| Android 版本 | Android 16 |
| API Level | 36 |
| minSdk | 26 |
| targetSdk | 36 |

## 测试范围

本项目主要覆盖以下内容：

- Text Note 新建、编辑、自动保存与数据持久化
- Checklist 新建、批量添加、勾选、取消勾选、删除与排序
- Note 新建、切换、重命名与删除
- Note 内容搜索及匹配高亮
- PIN / Pattern 锁定与解锁
- Open File / Export as File 文件操作
- Share、Settings、About 等基础入口可用性
- 前后台切换、锁屏恢复、横竖屏、最近任务清除等移动端专项场景
- ADB 安装、设备信息、进程、Activity、force-stop、Logcat 等验证
- 冷启动、内存、CPU 与 Monkey 稳定性测试

未对 Settings 内全部配置项进行深度展开，也未进行接口、服务端数据库或 JMeter 并发压测。Fossify Notes 本项目测试重点为本地 Android 客户端行为，不为展示工具而强行加入与项目形态不匹配的测试内容。

## 测试用例执行结果

共设计并执行 **276 条测试用例**。

| 指标 | 结果 |
|---|---:|
| 测试用例总数 | 276 |
| 已执行 | 276 |
| PASS | 276 |
| FAIL | 0 |
| 执行率 | 100% |
| 通过率 | 100% |

本轮测试未发现功能性 Bug 或异常结果。

## 移动端专项测试

除常规功能验证外，测试用例中同时覆盖了多项 Android 移动端专项场景，包括：

- 应用首次安装、卸载、重新安装与覆盖安装
- 前后台切换
- 最近任务清除后重新启动
- ADB 强制停止后恢复
- 横竖屏切换
- 锁屏 / 解锁
- 系统返回键
- 深色模式
- 字体与显示缩放
- 无网络环境
- Text Note 自动保存与冷启动数据恢复
- Checklist 状态持久化
- PIN / Pattern 锁定后的冷启动保护
- 文件选择器、导入与导出流程中断恢复

## ADB 测试

共完成 **13 项 ADB / 命令行验证**，主要包括：

- APK 安装
- 模拟设备型号查询
- Android 版本与 API Level 查询
- 应用版本与 SDK 信息查询
- 应用 PID 查询
- Launcher Activity 查询
- `force-stop` 强制停止
- ADB 启动应用并采集冷启动耗时
- `dumpsys meminfo` 内存信息检查
- `top` CPU 连续采样
- Logcat 日志检查
- Monkey 随机事件稳定性测试

测试过程中未观察到 `FATAL EXCEPTION`、Crash 或 ANR。

## 基础性能与稳定性测试

### 冷启动

连续执行 5 次冷启动：

| 次数 | TotalTime | WaitTime |
|---:|---:|---:|
| 1 | 612 ms | 614 ms |
| 2 | 431 ms | 435 ms |
| 3 | 511 ms | 513 ms |
| 4 | 414 ms | 416 ms |
| 5 | 310 ms | 312 ms |

- TotalTime 平均值：**455.6 ms**
- 最快：**310 ms**
- 最慢：**612 ms**

5 次均为 `LaunchState: COLD`，并成功进入 `MainActivity`。

### 内存

连续操作前后使用 `dumpsys meminfo` 进行观察：

| 阶段 | TOTAL PSS | TOTAL RSS | Native Heap PSS |
|---|---:|---:|---:|
| 操作前基线 | 29,140 KB | 135,048 KB | 11,308 KB |
| 连续操作后 | 67,468 KB | 181,388 KB | 35,236 KB |
| 返回主页面并空闲后 | 37,328 KB | 146,256 KB | 15,224 KB |

连续操作后内存明显增长，但退出额外页面并空闲后，PSS、Native Heap、Activity 与 AppContext 均明显回落。本轮测试中未观察到持续性内存增长，暂未发现明显内存泄漏迹象。

### CPU

通过 `top` 连续采样 15 次：

- 空闲采样：约 **0%**
- 操作期间 CPU 随负载波动
- 平均 CPU：约 **31.1%**
- 峰值 CPU：**68%**

连续进行文本输入、Note 切换、搜索及 Checklist 操作时，应用保持正常响应，未出现 ANR 或 Crash。

### Monkey 稳定性

执行：

```bash
adb shell monkey -p org.fossify.notes --throttle 100 --pct-syskeys 0 -v 500
```

结果：

- 成功注入 **500 个随机事件**
- Monkey 正常结束
- 未出现 Crash
- 未出现 ANR
- 未出现 FATAL EXCEPTION
- 测试结束后 App 可正常打开和操作
- 原有 Note / Checklist 数据仍存在

其中 3 个 flip 事件因模拟器 `/dev/input/event0` 权限限制而被丢弃，该现象属于模拟器输入设备权限限制，不判定为应用缺陷。

## 测试结论

在本次 Android Emulator、Android 16 / API 36 的测试环境下，Fossify Notes 核心功能、移动端专项场景、ADB 验证以及基础性能与稳定性测试均完成。

共执行 **276 条测试用例，全部通过**；完成 **13 项 ADB 验证**；完成冷启动、内存、CPU 及 Monkey 稳定性测试。测试过程中未发现功能性 Bug、Crash 或 ANR。

本项目结论仅适用于本次模拟器及既定测试条件。由于未使用真实 Android 物理设备，因此未覆盖不同品牌、不同 ROM、不同硬件配置下的真实设备兼容性表现。

## 项目文档

```text
Fossify-Notes-Mobile-Test/
├── README.md
├── 01-测试需求分析/
│   └── Fossify_Notes移动端测试需求分析.xlsx
├── 02-测试用例/
│   └── Fossify_Notes移动端测试用例.xlsx
├── 03-ADB测试/
│   └── Fossify_Notes_ADB测试记录.xlsx
├── 04-性能与稳定性/
│   └── Fossify_Notes_基础性能与稳定性测试.xlsx
└── 05-测试报告/
    └── Fossify_Notes移动端软件测试报告.docx
```

## 项目收获

通过本项目完整实践了 Android 移动端测试流程，包括测试范围分析、测试用例设计与执行、移动端专项测试、ADB 常用命令、Logcat 日志检查、冷启动耗时采集、内存与 CPU 观察、Monkey 稳定性测试以及最终测试报告输出。

该项目与 Web 测试项目形成互补，重点体现移动端测试思路、Android 环境下的异常恢复与数据持久化验证，以及 ADB 在实际测试工作中的使用。
