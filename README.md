# 📱 手机芯片底层技术栈收集

专注手机/芯片底层技术积累，紧跟业界前沿。

---

## 📂 目录结构

- [Linux 内核](#linux-内核)
- [ARM Trusted Firmware](#arm-trusted-firmware)
- [ARM 架构与指令集](#arm-架构与指令集)
- [编译器与工具链](#编译器与工具链)
- [调度与功耗优化](#调度与功耗优化)
- [移动端 AI/ML](#移动端-aiml)
- [Android 底层](#android-底层)
- [安全与 TrustZone](#安全与-trustzone)
- [软件架构参考](#软件架构参考)
- [工具与模拟器](#工具与模拟器)
- [学习资源](#学习资源)

---

## Linux 内核

### 核心仓库

| 项目 | Stars | 说明 |
|------|-------|------|
| [torvalds/linux](https://github.com/torvalds/linux) | ⭐222K | Linux 内核主分支 |
| [linux-stable](https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git) | - | 稳定分支 |
| [ARM64 Linux](https://git.kernel.org/pub/scm/linux/kernel/git/arm64/linux.git) | - | ARM64 架构专用分支 |

### 关键子系统

- **调度器**: `kernel/sched/` - CFS, EEVDF, EAS
- **CPUFreq**: `drivers/cpufreq/` - schedutil, cpuidle
- **热管理**: `drivers/thermal/` - thermal governor
- **电源管理**: `kernel/power/` - suspend, hibernate
- **内存管理**: `mm/` - 内存分配, 页面管理

### 内核新特性跟踪

| 项目 | Stars | 说明 |
|------|-------|------|
| [0voice/kernel_new_features](https://github.com/0voice/kernel_new_features) | ⭐1.8K | Linux 内核新特性深挖 |
| [linux-test-project/ltp](https://github.com/linux-test-project/ltp) | ⭐2.5K | Linux 测试项目 |

---

## ARM Trusted Firmware

### 官方仓库

| 项目 | Stars | 说明 |
|------|-------|------|
| [ARM-software/arm-trusted-firmware](https://github.com/ARM-software/arm-trusted-firmware) | ⭐2.1K | TF-A 官方镜像，Secure Monitor |
| [ARM-software/tf-a-tests](https://github.com/ARM-software/tf-a-tests) | - | TF-A 测试套件 |
| [ARM-software/arm-trusted-firmware/wiki](https://github.com/ARM-software/arm-trusted-firmware/wiki) | - | 官方文档 |

### ATF 关键组件

- **BL1**: AP Trusted ROM
- **BL2**: Trusted Boot Firmware
- **BL31**: EL3 Runtime Software
- **BL32**: Secure-EL1 Payload (OP-TEE)
- **BL33**: Non-trusted Firmware (U-Boot/UEFI)

---

## ARM 架构与指令集

### 官方规范与库

| 项目 | Stars | 说明 |
|------|-------|------|
| [ARM-software/abi-aa](https://github.com/ARM-software/abi-aa) | ⭐1.2K | ARM ABI 规范 |
| [ARM-software/optimized-routines](https://github.com/ARM-software/optimized-routines) | ⭐685 | ARM 优化库函数 |
| [ARM-software/ComputeLibrary](https://github.com/ARM-software/ComputeLibrary) | ⭐3.1K | CPU/GPU 计算库 (SIMD/NEON) |

### CMSIS 系列

| 项目 | Stars | 说明 |
|------|-------|------|
| [ARM-software/CMSIS_5](https://github.com/ARM-software/CMSIS_5) | ⭐1.5K | CMSIS v5 开发库 |
| [ARM-software/CMSIS-DSP](https://github.com/ARM-software/CMSIS-DSP) | ⭐953 | DSP 计算库 |
| [ARM-software/Arm-2D](https://github.com/ARM-software/Arm-2D) | ⭐396 | 2D 图形库 (Cortex-M) |

### SIMD/NEON

| 项目 | Stars | 说明 |
|------|-------|------|
| [simd-everywhere/simde](https://github.com/simd-everywhere/simde) | ⭐2.9K | SIMD 指令集跨平台实现 |

---

## 编译器与工具链

### 主流编译器

| 项目 | Stars | 说明 |
|------|-------|------|
| [llvm/llvm-project](https://github.com/llvm/llvm-project) | ⭐37K | LLVM 编译器，ARM 后端优化 |
| [gcc-mirror/gcc](https://github.com/gcc-mirror/gcc) | ⭐10K | GCC 编译器 |
| [ARM-software/arm-toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain) | - | ARM 官方工具链 |

### 嵌入式开发

| 项目 | Stars | 说明 |
|------|-------|------|
| [xpack-dev-tools/arm-none-eabi-gcc-xpack](https://github.com/xpack-dev-tools/arm-none-eabi-gcc-xpack) | ⭐266 | ARM 嵌入式 GCC 发行版 |
| [libopencm3/libopencm3](https://github.com/libopencm3/libopencm3) | ⭐3.5K | ARM Cortex-M 开源库 |

---

## 调度与功耗优化

### 内核调度器

| 组件 | 路径 | 说明 |
|------|------|------|
| CFS | `kernel/sched/fair.c` | 完全公平调度器 |
| EEVDF | `kernel/sched/fair.c` | 替代 CFS 的新调度器 (6.12+) |
| EAS | `kernel/sched/energy.c` | 能量感知调度 |
| DL | `kernel/sched/deadline.c` | 实时调度器 |
| RT | `kernel/sched/rt.c` | 实时调度器 |

### CPUFreq 调频

| Governor | 文件 | 说明 |
|----------|------|------|
| schedutil | `cpufreq_schedutil.c` | 调度器驱动调频 |
| ondemand | `cpufreq_ondemand.c` | 按需调频 |
| conservative | `cpufreq_conservative.c` | 保守调频 |
| powersave | - | 省电模式 |
| performance | - | 性能模式 |

### CPUIdle 休闲

| Governor | 说明 |
|----------|------|
| ladder | 阶梯式休闲 |
| menu | 菜单式休闲 |
| TEO | Timer Events Oriented (6.6+) |

### 参考项目

| 项目 | Stars | 说明 |
|------|-------|------|
| [auto-cpufreq](https://github.com/AdnanHodzic/auto-cpufreq) | ⭐7.4K | 用户态自动调频 |

---

## 移动端 AI/ML

### 端侧推理

| 项目 | Stars | 说明 |
|------|-------|------|
| [ARM-software/ML-KWS-for-MCU](https://github.com/ARM-software/ML-KWS-for-MCU) | ⭐1.2K | 关键词识别 (Cortex-M) |
| [cactus-compute/cactus](https://github.com/cactus-compute/cactus) | ⭐4.4K | 移动端 AI 引擎 |
| [tensorflow/tflite-micro](https://github.com/tensorflow/tflite-micro) | - | TensorFlow Lite Micro |

### AI 辅助调度趋势

- 高通 AI-Assisted Scheduling (8 Elite Gen 2)
- 联发科 CorePilot 7.0 (天玑 9500)
- 苹果 Apple Intelligence 调度

---

## Android 底层

### 系统架构

| 项目 | Stars | 说明 |
|------|-------|------|
| [AOSP](https://source.android.com/) | - | Android 开源项目 |
| [android/architecture-samples](https://github.com/android/architecture-samples) | ⭐45K | 架构示例 |

### HAL/驱动层

- **AudioHAL**: 音频硬件抽象层
- **CameraHAL**: 相机硬件抽象层
- **PowerHAL**: 电源管理 HAL
- **ThermalHAL**: 热管理 HAL

### Vendor 实现

- 高通 CAF (Code Aurora Forum)
- 联发科 MTK
- 华为 Kirin (部分开源)

---

## 安全与 TrustZone

### TEE 框架

| 项目 | Stars | 说明 |
|------|-------|------|
| [OP-TEE/optee_os](https://github.com/OP-TEE/optee_os) | ⭐1.4K | 开源 TEE OS |
| [OP-TEE/optee_client](https://github.com/OP-TEE/optee_client) | - | TEE 客户端库 |

### 安全机制

- **TrustZone**: ARM 硬件隔离
- **Secure Boot**: 安全启动链
- **Keymaster**: 密钥管理
- **Gatekeeper**: 认证管理

---

## 软件架构参考

> 现代化软件项目架构设计参考，虽然不是直接的芯片底层代码，但其设计模式、性能优化策略、跨平台架构对 Kirin 系统设计有重要参考价值。

### OpenClaw - 多通道 AI 网关

| 属性 | 内容 |
|------|------|
| **项目** | [openclaw/openclaw](https://github.com/openclaw/openclaw) ⭐ |
| **版本** | 2026.3.2 |
| **定位** | Multi-channel AI Gateway (多通道 AI 网关) |
| **技术栈** | TypeScript/Node.js 22+ |
| **协议** | MIT |

#### 核心架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Layer (跨平台)                     │
│  iOS App │ Android App │ macOS App │ TUI Terminal          │
└───────────────────────────┬─────────────────────────────────┘
                            │ ACP Protocol (JSON-RPC over WS)
┌───────────────────────────┼─────────────────────────────────┐
│                    Gateway Layer (Node.js)                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  43 Extensions (Discord/Telegram/Signal/Feishu/...)  │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────┼─────────────────────────────────┐
│                    Skill Layer (54 Skills)                   │
│  Built-in Skills │ User Skills │ Skill Registry (clawhub)   │
└─────────────────────────────────────────────────────────────┘
```

#### 技术亮点 - 对 Kirin 的参考价值

| 设计模式 | OpenClaw 实现 | Kirin 应用场景 |
|----------|---------------|----------------|
| **多通道统一抽象** | 43 个消息通道统一接口 | 手机/平板/车机跨设备消息同步 |
| **插件热加载** | 运行时技能安装/更新 | 系统功能免重启更新 |
| **本地向量检索** | sqlite-vec + LanceDB | 端侧 AI 记忆系统 |
| **ACP 协议** | Agent Client Protocol | 分布式 Agent 协作标准 |
| **mDNS 设备发现** | @homebridge/ciao | 近场设备自动发现配对 |

#### 高性能技术栈

| 组件 | 选型 | 性能亮点 |
|------|------|----------|
| **构建** | tsdown | 比 tsc 快 10x+ |
| **图像处理** | sharp (libvips) | 原生 C 加速 |
| **本地 LLM** | node-llama-cpp | 支持 NPU 加速潜力 |
| **HTTP 客户端** | undici | HTTP/2 + 连接池优化 |
| **向量存储** | sqlite-vec | 纯 SQLite 实现，无需额外服务 |

#### 扩展系统 (43 Extensions)

按技术类型分类：

| 类型 | 扩展示例 | 技术深度 |
|------|----------|----------|
| **即时通讯** | discord, telegram, signal, feishu, whatsapp | 协议逆向、E2E 加密 |
| **企业协作** | msteams, slack, mattermost | Graph API、Webhook |
| **语音通话** | voice-call | WebRTC + Opus + ICE 穿透 |
| **设备配对** | device-pair | mDNS/Bonjour 服务发现 |
| **记忆系统** | memory-core, memory-lancedb | 向量 RAG、本地优先 |

#### 安全设计参考

- **Signal Protocol**: 端到端加密实现 (`extensions/signal`)
- **设备配对**: mDNS + 密钥交换 (`extensions/device-pair`)
- **沙箱隔离**: 技能运行时隔离

#### 源码关键路径

```
openclaw/
├── src/
│   ├── gateway/      # 网关核心 - 消息路由
│   ├── agent/        # Agent 运行时
│   ├── protocol/     # ACP 协议实现
│   └── daemon/       # 守护进程
├── extensions/       # 43 个通道扩展
├── skills/           # 54 个内置技能
├── apps/
│   ├── ios/          # Swift + SwiftUI
│   ├── android/      # Kotlin + Compose
│   └── macos/        # Swift + AppKit
└── docs/             # Mintlify 文档
```

#### 对 Kirin 平台的具体启示

**调度优化参考**:
- 消息队列设计可优化 EAS 任务分发
- TUI 低功耗模式可借鉴终端场景调度
- 多进程通信模式可优化 Binder 调用

**功耗优化参考**:
- 本地 LLM 推理的 CPU/GPU 负载均衡
- WebSocket 长连接的休眠策略
- 插件按需加载减少内存占用

**安全设计参考**:
- 设备配对流程可作为 TrustZone 参考
- E2E 加密实现可借鉴到安全通信

**AI 能力参考**:
- 端侧 LLM + 向量检索架构
- NPU 调度策略优化
- 本地优先的 AI 设计理念

---

## 工具与模拟器

### 模拟器

| 项目 | Stars | 说明 |
|------|-------|------|
| [unicorn-engine/unicorn](https://github.com/unicorn-engine/unicorn) | ⭐8.8K | CPU 模拟器框架 |
| [qemu/qemu](https://github.com/qemu/qemu) | ⭐10K+ | 全系统模拟器 |

### 逆向分析

| 项目 | Stars | 说明 |
|------|-------|------|
| [capstone-engine/capstone](https://github.com/capstone-engine/capstone) | ⭐8.5K | 反汇编框架 |
| [keystone-engine/keystone](https://github.com/keystone-engine/keystone) | - | 汇编框架 |

### 调试工具

| 项目 | Stars | 说明 |
|------|-------|------|
| [stlink-org/stlink](https://github.com/stlink-org/stlink) | ⭐5K | STM32 调试工具 |
| [Dr-Noob/cpufetch](https://github.com/Dr-Noob/cpufetch) | ⭐2.1K | CPU 信息获取 |

---

## 学习资源

### 书籍推荐

- 《Linux Kernel Development》- Robert Love
- 《Understanding the Linux Kernel》- Bovet & Cesati
- 《ARM System Developer's Guide》- Andrew Sloss
- 《Computer Architecture: A Quantitative Approach》- Hennessy & Patterson

### 在线资源

| 资源 | 链接 | 说明 |
|------|------|------|
| LWN.net | https://lwn.net | Linux 内核新闻 |
| Kernel Newbies | https://kernelnewbies.org | 内核入门 |
| ARM Developer | https://developer.arm.com | ARM 官方文档 |
| Linux Kernel Documentation | https://www.kernel.org/doc/ | 内核文档 |

### 竞品技术跟踪

- **高通**: Snapdragon 8 Elite Gen 5 (Oryon 架构, AI 调度)
- **联发科**: 天玑 9500 (All Big Core, CorePilot 7.0)
- **苹果**: A20 (2nm, WMCM 封装)
- **华为**: Kirin (自主调度优化)

---

## 📅 更新日志

- **2026-03-12**: 添加 OpenClaw 软件架构参考，包含 43 扩展 + 54 技能深度分析

---

## 🤝 贡献

欢迎提交 PR 补充优质资源！

---

> 🦞 由 [OpenClaw](https://github.com/openclaw/openclaw) 自动维护
