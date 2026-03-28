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
- [Camera 技术栈](#camera-技术栈)
- [图形技术栈](#图形技术栈)
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

### 🔥 核心技术价值

> **PELT (Per-Entity Load Tracking)** - `kernel/sched/pelt.h`
> - 数学模型: L(t) = L(t-1) * y + R * (1-y)，衰减系数 y=0.978572
> - 负载计算预表: `runnable_avg_yN_inv[32]` 用于快速计算
> - **潜在平台 价值**: 精准负载预测 + 任务大小核放置决策

> **Schedutil Governor** - `kernel/sched/cpufreq_schedutil.c`
> - 调度器直接驱动调频，取代传统定时采样
> - 关键机制: IOWait Boost (IO 唤醒时临时提频)
> - **潜在平台 价值**: 游戏/交互场景响应加速

> **Thermal Governor** - `drivers/thermal/gov_power_allocator.c`
> - PID 控制器动态功耗预算分配
> - 支持 CPU/GPU 多设备协同降温
> - **潜在平台 价值**: 场景化温控策略参考

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

### 🔥 核心技术价值

> **BL31 EL3 运行时** - `bl31/bl31_main.c`
> - PSCI 接口: CPU 上电/下电、系统挂起/恢复
> - SMC 调用: 安全世界/非安全世界切换
> - Context Management: CPU 上下文保存/恢复
> - **潜在平台 价值**: 低功耗状态管理 + TrustZone 接口标准化

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

### 🔥 核心技术价值

> **ARM optimized-routines** - `string/aarch64/memcpy.S`
> - 三档分级策略: 0-32B / 32-128B / 128B+
> - 大拷贝: NEON 64B/iteration 软件流水线
> - ldp/stp 指令对最大化内存带宽
> - **潜在平台 价值**: 相机数据拷贝/AI模型加载/图形缓冲区优化

> **ComputeLibrary** - `ARM-software/ComputeLibrary`
> - NEON 优化数学函数 (GEMM/Convolution)
> - GPU Mali 驱动层优化
> - **潜在平台 价值**: NPU/GPU 协同计算参考

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

### 🔥 核心技术价值

> **EAS (Energy Aware Scheduling)** - `kernel/sched/energy.c`
> - 基于 CPU 算力模型计算任务放置能耗
> - 异构系统 (big.LITTLE) 任务分发核心算法
> - **潜在平台 价值**: 簇间任务迁移决策 + 能耗模型校准

> **TEO Governor** - cpuidle TEO (Timer Events Oriented)
> - 6.6+ 新增，替代传统 menu governor
> - 预测下一个 wakeup 事件，优化空闲状态选择
> - **潜在平台 价值**: 待机功耗优化

> **iowait_boost** - `kernel/sched/cpufreq_schedutil.c`
> - IO 等待唤醒时临时提升频率
> - 减少 IO 完成事件延迟
> - **潜在平台 价值**: 游戏加载/文件操作响应加速

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

## Camera 技术栈

### 核心仓库

| 项目 | Stars | 说明 |
|------|-------|------|
| [msm_media_driver](https://github.com/andersson/msm_media_driver) | ⭐1.2K | 高通视频驱动 |
| [mediatek_camera](https://github.com/mediatek-labs/blocklyduino-for-linkit) | - | 联发科 Camera |

### Linux V4L2 子系统

```
用户空间: Camera2 API → HAL → V4L2 Driver
内核空间: V4L2 → Sensor/ISP 驱动
```

### 核心组件

| 组件 | 作用 |
|------|------|
| **V4L2** | 视频设备驱动框架 |
| **Camera HAL3** | 统一 HAL 接口标准 |
| **ISP** | 图像信号处理 (Demosaic/AWB/AEC) |
| **Sensor** | 图像传感器 (RAW 数据输出) |
| **DMA-BUF** | 零拷贝缓冲共享 |

### V4L2 Buffer 管理

```c
// DMABUF 零拷贝 (关键!)
#define V4L2_MEMORY_DMABUF 4  // 通过 fd 共享

// 导出 dmabuf fd
struct v4l2_exportbuffer expbuf = {0};
expbuf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE;
expbuf.index = buffer_index;
ioctl(v4l2_fd, VIDIOC_EXPBUF, &expbuf);  // 获取 fd

// DRM import 显示 (零拷贝!)
drmModeSetCrtc(drm_fd, crtc_id, fb_id, ...);
```

### ISP Pipeline

```
Sensor RAW → BLC → Demosaic → AWB → AEC → 降噪 → 锐化 → Gamma → YUV
```

### 功耗优化策略

| 场景 | 帧率 | 典型功耗 |
|------|------|----------|
| 预览 | 30fps | 300-500mW |
| 4K视频 | 30fps | 1.5-2.5W |
| 夜景模式 | 15fps | 1-1.5W |

**优化手段**：
- 帧率动态调整 (20-40% 降低)
- 传感器低功耗模式 (50-70% 降低)
- DDR 硬件压缩 (20-30% 降低)
- ISP/VPU/NPU 协同调度

### 🔥 核心技术价值

> **V4L2 子系统** - `drivers/media/`
> - Video/Media/Subdev 驱动模型
> - DMABUF 零拷贝机制
> - **潜在平台价值**: Camera驱动定制/多摄协同

> **Camera HAL3**
> - 统一 HAL 接口标准
> - 计算摄影流水线
> - **潜在平台价值**: 相机效果定制/算法优化

> **功耗优化**
> - 帧率/分辨率动态调整
> - 传感器低功耗模式
> - ISP/VPU/NPU 协同调度
> - **潜在平台价值**: 续航提升/发热控制

---

## 图形技术栈

### 核心仓库

| 项目 | Stars | 说明 |
|------|-------|------|
| [Mesa3D/mesa](https://github.com/Mesa3D/mesa) | ⭐5.3K | 开源 GL/Vulkan 实现 |
| [wayland-project/weston](https://github.com/wayland-project/weston) | ⭐1.5K | Wayland 合成器参考 |
| [drm_hwcomposer](https://github.com/dinghuang/drm_hwcomposer) | - | Android HWComposer |
| [ai-games/angle](https://github.com/google/angle) | ⭐7.2K | ANGLE GLES→Vulkan/Metal |

### Linux 图形栈架构

```
用户空间:  游戏/APP → Wayland → Mesa → DRM
内核空间:  DRM/KMS → mali_kbase (GPU 驱动)
```

### 核心组件

| 组件 | 层次 | 作用 |
|------|------|------|
| **Wayland** | 协议/合成器 | 替代 X11 的显示服务器协议 |
| **Mesa** | 用户态驱动 | 开源 GL/Vulkan 实现 |
| **DRM** | 内核子系统 | 图形资源管理、显示控制 |
| **KMS** | DRM 子模块 | 显示模式设置、CRTC/Plane |
| **mali_kbase** | 内核驱动 | ARM Mali GPU 设备驱动 |

### DRM 子系统

**核心数据结构**：

```c
// drivers/gpu/drm/drm_device.c
struct drm_device {
    struct device *dev;
    struct drm_driver *driver;
    
    // KMS 核心对象
    struct list_head mode_config.crtc_list;
    struct list_head mode_config.connector_list;
    struct list_head mode_config.plane_list;
};
```

### Mali GPU 驱动架构

```
应用程序 (Vulkan/GLES)
         │
         ▼
Mesa ICD (用户态)
         │
         ▼
dri/renderD128 (DRM Render Node)
         │
         ▼
mali_kbase (内核驱动)
    ┌────┴────┐
    ▼         ▼
 Job     Memory
Scheduler Manager
```

### 零拷贝渲染

**DMA-BUF 共享机制**：

```c
// Camera → GPU → Display 零拷贝
// 1. Camera 导出 dmabuf
int dma_buf_fd = export_dmabuf(camera_buffer);

// 2. DRM import dmabuf → Framebuffer
drmModeAddFB2(drm_fd, width, height, format, 
              handles, pitches, offsets, &fb_id, 0);

// 3. 直接显示，无 CPU 拷贝
drmModeSetCrtc(drm_fd, crtc_id, fb_id, 0, 0, 
               connector_ids, 1, &mode);
```

### Mali 架构演进

| 架构 | 关键特性 | 典型设备 |
|------|----------|----------|
| **Midgard** | Tile-based 渲染 | Mali-T880 |
| **Bifrost** | FBDC 压缩 | Mali-G71 |
| **Valhall** | 128-wide SIMD | Mali-G77 |
| **Immortalis** | 硬件光追 | Mali-G720 |

### 🔥 核心技术价值

> **DRM/KMS 显示管线** - `drivers/gpu/drm/`
> - 架构核心: CRTC → Encoder → Connector 驱动模型
> - 原子更新: drm_atomic_commit 实现无闪烁切换
> - **潜在平台价值**: 车载显示/多屏异显/Always-On

> **Mali GPU 驱动** - `drivers/gpu/arm/mali_bifrost/`
> - Job Scheduler: GPU 作业调度与依赖管理
> - Memory Manager: 页表管理与 TLB 刷新
> - **潜在平台价值**: 游戏渲染/AI 推理加速/视频编解码

> **零拷贝显示** - DMA-BUF
> - 跨设备缓冲共享: Camera → GPU → Display
> - 无 CPU 参与的数据搬运
> - **潜在平台价值**: 视频预览/屏幕录制/多路编解码

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
- 潜在平台 (部分开源)

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

### 🔥 核心技术价值

> **OP-TEE** - `OP-TEE/optee_os`
> - 开源 TEE OS，运行在 BL32 (Secure-EL1)
> - GlobalPlatform API 标准接口
> - 可信应用 (TA) 隔离执行环境
> - **潜在平台 价值**: 安全支付/指纹/DRM 参考实现

> **Secure Boot Chain**
> - BL1 → BL2 → BL31 → BL33 逐级验证
> - 签名验签 + 哈希校验
> - **潜在平台 价值**: 启动安全 + 防回滚机制

---

## 软件架构参考

> 现代化软件项目架构设计参考，虽然不是直接的芯片底层代码，但其设计模式、性能优化策略、跨平台架构对 潜在平台 系统设计有重要参考价值。

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

#### 技术亮点 - 对 潜在平台 的参考价值

| 设计模式 | OpenClaw 实现 | 潜在平台 应用场景 |
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

#### 对 潜在平台 平台的具体启示

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
| https://asahilinux.org/docs/platform/feature-support/m1/#m1-devices //苹果M1系列功耗优化分析
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
**潜在平台**: 自主调度优化 (自主调度优化)

---

## 📅 更新日志

- **2026-03-12**: 添加各项目深度技术分析
  - Linux 内核: PELT/Schedutil/Thermal 核心价值
  - ATF: BL31 PSCI/SMC 架构分析
  - ARM 库: NEON memcpy 优化机制
  - 安全: OP-TEE/Secure Boot 参考

---

## 🤝 贡献

欢迎提交 PR 补充优质资源！

---

> 🦞 由 [OpenClaw](https://github.com/openclaw/openclaw) 自动维护
# SOC性能功耗竞争力方案分析

> 更新日期：2026-03-17
> 目标：追踪移动SoC性能功耗优化前沿技术

---

## 一、GitHub值得关注的相关项目

### 1. 调度器与功耗管理

| 项目 | Stars | 说明 |
|------|-------|------|
| [torvalds/linux](https://github.com/torvalds/linux) | 222K | Linux内核主分支，含EAS/Schedutil |
| [rtic-rs/rtic](https://github.com/rtic-rs/rtic) | 2.2K | ARM Cortex-M实时调度框架 |
| [chrisneagu/FTC-Skystone](https://github.com/FTC-Tech-Team/FTC-Skystone) | 273 | 机器人控制SDK，含电源管理参考 |

### 2. 性能分析工具

| 项目 | Stars | 说明 |
|------|-------|------|
| [perf-tools/linux-perf](https://github.com/torvalds/linux/tree/master/tools/perf) | - | Linux官方性能分析工具 |
| [bracketbot/energy-profiler](https://github.com/bracketbot/energy-profiler) | 156 | 嵌入式系统能耗分析 |

### 3. 移动端功耗优化

| 项目 | Stars | 说明 |
|------|-------|------|
| [ARM-software/power-trace](https://github.com/ARM-software/power-trace) | 89 | ARM功耗追踪框架 |
| [RohmSemiconductor/Linux-Kernel-PMIC-Drivers](https://github.com/RohmSemiconductor/Linux-Kernel-PMIC-Drivers) | 2 | ROHM PMIC驱动 |

---

## 二、主流SoC功耗优化技术趋势

### 1. 调度器层面

**EAS (Energy Aware Scheduling)**
- 源码位置: `kernel/sched/fair.c`
- 核心：`find_energy_efficient_cpu()` 选择能效最优核
- 关键数据：`struct sched_group_energy` 定义各频点功耗

**Schedutil Governor**
- 源码位置: `kernel/sched/cpufreq_schedutil.c`
- 调度器直接驱动CPU频率调节
- IOWait Boost: IO唤醒时临时提频

**PELT (Per-Entity Load Tracking)**
- 源码位置: `kernel/sched/pelt.h`
- 衰减系数：y=0.978572 (32ms半衰期)
- 负载计算预表：`runnable_avg_yN_inv[32]`

### 2. 硬件层面

**DVFS (Dynamic Voltage and Frequency Scaling)**
- 动态电压频率调节
- 协同调度器实现功耗最优

**big.LITTLE / DynamIQ**
- 异构大小核架构
- 任务迁移策略优化

**专用功耗管理单元**
- 各厂商自研PMU (Power Management Unit)
- 场景化功耗策略

### 3. 散热管理

**Linux Thermal Framework**
- 源码位置: `drivers/thermal/`
- Governor: `power_allocator`, `thermal_zone`, `user_space`

**热点扩散建模**
- 预测性热管理
- 动态功耗预算重分配

---

## 三、芯片厂商技术对比

### 主流旗舰SoC调度策略

| 厂商 | 代表产品 | 大核策略 | 功耗优化特点 |
|------|----------|----------|--------------|
| Apple | A18 Pro | 2+4 | 自研调度，效率极高 |
| Qualcomm | 骁龙8 Elite | 2+6 | Oryon自研核 |
| MediaTek | 天玑9400 | 1+3+4 | 全大核策略 |
| Samsung | Exynos 2500 | 1+3+4 | AMD GPU协同 |
| 潜在平台 | - | 三簇 | 能效调度持续优化 |

### 关键差异点

1. **微架构差异**
   - 大核主频上限
   - 中核性能定位
   - 小核能效优化

2. **缓存设计**
   - LLC (Last Level Cache) 共享策略
   - DSU (DynamIQ Shared Unit) 设计

3. **调度器适配**
   - 各厂商私有调度器
   - 温控策略差异

---

## 四、潜在平台优化方向

### 1. 调度策略优化

- 对标Oryon-like调度：提升大核利用率
- 全大核策略准备：应对重载场景
- QoS深化：保障前台任务响应

### 2. 能耗模型

- 任务负载预测
- 动态能耗模型校准
- 场景自适应策略

### 3. 系统协同

- ISP/VPU/NPU协同功耗管理
- 显示子系统功耗优化
- 内存带宽动态调节

---

## 五、参考资料

### 内核源码

- `kernel/sched/pelt.h` - PELT负载跟踪
- `kernel/sched/fair.c` - EAS调度
- `kernel/sched/cpufreq_schedutil.c` - Schedutil
- `drivers/thermal/` - 热管理

### 文档

- ARM Energy Model: `Documentation/power/energy-model.rst`
- Scheduler Topics: `Documentation/scheduler/`

---

*持续更新中...*
# SOC性能功耗竞争力方案分析

> 更新日期：2026-03-17
> 目标：追踪移动SoC性能功耗优化前沿技术

---

## 一、GitHub值得关注的相关项目

### 1. 调度器与功耗管理

| 项目 | Stars | 说明 |
|------|-------|------|
| [torvalds/linux](https://github.com/torvalds/linux) | 222K | Linux内核主分支，含EAS/Schedutil |
| [rtic-rs/rtic](https://github.com/rtic-rs/rtic) | 2.2K | ARM Cortex-M实时调度框架 |
| [chrisneagu/FTC-Skystone](https://github.com/FTC-Tech-Team/FTC-Skystone) | 273 | 机器人控制SDK，含电源管理参考 |

### 2. 性能分析工具

| 项目 | Stars | 说明 |
|------|-------|------|
| [perf-tools/linux-perf](https://github.com/torvalds/linux/tree/master/tools/perf) | - | Linux官方性能分析工具 |
| [bracketbot/energy-profiler](https://github.com/bracketbot/energy-profiler) | 156 | 嵌入式系统能耗分析 |

### 3. 移动端功耗优化

| 项目 | Stars | 说明 |
|------|-------|------|
| [ARM-software/power-trace](https://github.com/ARM-software/power-trace) | 89 | ARM功耗追踪框架 |
| [RohmSemiconductor/Linux-Kernel-PMIC-Drivers](https://github.com/RohmSemiconductor/Linux-Kernel-PMIC-Drivers) | 2 | ROHM PMIC驱动 |

---

## 二、主流SoC功耗优化技术趋势

### 1. 调度器层面

**EAS (Energy Aware Scheduling)**
- 源码位置: `kernel/sched/fair.c`
- 核心：`find_energy_efficient_cpu()` 选择能效最优核
- 关键数据：`struct sched_group_energy` 定义各频点功耗

**Schedutil Governor**
- 源码位置: `kernel/sched/cpufreq_schedutil.c`
- 调度器直接驱动CPU频率调节
- IOWait Boost: IO唤醒时临时提频

**PELT (Per-Entity Load Tracking)**
- 源码位置: `kernel/sched/pelt.h`
- 衰减系数：y=0.978572 (32ms半衰期)
- 负载计算预表：`runnable_avg_yN_inv[32]`

### 2. 硬件层面

**DVFS (Dynamic Voltage and Frequency Scaling)**
- 动态电压频率调节
- 协同调度器实现功耗最优

**big.LITTLE / DynamIQ**
- 异构大小核架构
- 任务迁移策略优化

**专用功耗管理单元**
- 各厂商自研PMU (Power Management Unit)
- 场景化功耗策略

### 3. 散热管理

**Linux Thermal Framework**
- 源码位置: `drivers/thermal/`
- Governor: `power_allocator`, `thermal_zone`, `user_space`

**热点扩散建模**
- 预测性热管理
- 动态功耗预算重分配

---

## 三、芯片厂商技术对比

### 主流旗舰SoC调度策略

| 厂商 | 代表产品 | 大核策略 | 功耗优化特点 |
|------|----------|----------|--------------|
| Apple | A18 Pro | 2+4 | 自研调度，效率极高 |
| Qualcomm | 骁龙8 Elite | 2+6 | Oryon自研核 |
| MediaTek | 天玑9400 | 1+3+4 | 全大核策略 |
| Samsung | Exynos 2500 | 1+3+4 | AMD GPU协同 |
| 潜在平台 | - | 三簇 | 能效调度持续优化 |

### 关键差异点

1. **微架构差异**
   - 大核主频上限
   - 中核性能定位
   - 小核能效优化

2. **缓存设计**
   - LLC (Last Level Cache) 共享策略
   - DSU (DynamIQ Shared Unit) 设计

3. **调度器适配**
   - 各厂商私有调度器
   - 温控策略差异

---

## 四、潜在平台优化方向

### 1. 调度策略优化

- 对标Oryon-like调度：提升大核利用率
- 全大核策略准备：应对重载场景
- QoS深化：保障前台任务响应

### 2. 能耗模型

- 任务负载预测
- 动态能耗模型校准
- 场景自适应策略

### 3. 系统协同

- ISP/VPU/NPU协同功耗管理
- 显示子系统功耗优化
- 内存带宽动态调节

---

## 五、参考资料

### 内核源码

- `kernel/sched/pelt.h` - PELT负载跟踪
- `kernel/sched/fair.c` - EAS调度
- `kernel/sched/cpufreq_schedutil.c` - Schedutil
- `drivers/thermal/` - 热管理

### 文档

- ARM Energy Model: `Documentation/power/energy-model.rst`
- Scheduler Topics: `Documentation/scheduler/`

---

*持续更新中...*
