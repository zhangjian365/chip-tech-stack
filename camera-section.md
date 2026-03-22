# Camera 技术栈

## 目录
- [为什么有价值](#为什么有价值-痛点分析)
- [架构设计](#架构设计)
- [核心实现](#核心实现)
- [功耗优化](#功耗优化-单独章节)

---

## 为什么有价值 - 痛点分析

### 1. 移动端Camera场景挑战

**痛点**：
- 视频通话/直播需要30fps+流畅度
- 计算摄影需要多帧合成（夜景/HDR）
- AI场景识别需要实时处理

**价值**：掌握Camera栈才能优化拍摄体验和功耗

### 2. 异构计算协同

**痛点**：
- ISP负责基础图像处理
- NPU负责AI场景检测
- GPU负责渲染预览

**价值**：理解数据流才能做系统级优化

### 3. 功耗是核心竞争力

**痛点**：
- Camera是Top3耗电场景
- 长时间视频录制发热严重
- 待机时快速启动需求

**价值**：功耗优化直接决定用户体验

---

## 架构设计

### Android Camera 整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                     应用层 (Apps)                            │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Camera2 API / CameraX                  │    │
│  │  (应用调用接口)                                      │    │
│  └──────────────────────┬──────────────────────────────┘    │
│                         ▼                                    │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           Camera Framework (Java)                   │    │
│  │  CameraService / CameraProvider                     │    │
│  └──────────────────────┬──────────────────────────────┘    │
│                         ▼                                    │
├─────────────────────────────────────────────────────────────┤
│                     HAL 层 (C++)                             │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Camera HAL (libcameraservice.so)         │    │
│  │  - camera_module_t                                  │    │
│  │  - camera_device_t                                  │    │
│  │  - camera3_stream_t                                 │    │
│  └──────────────────────┬──────────────────────────────┘    │
│                         ▼                                    │
├─────────────────────────────────────────────────────────────┤
│                     厂商实现 (Vendor)                         │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   ISP       │  │   Sensor    │  │     VPU/NPU        │ │
│  │ 图像信号处理 │  │   传感器    │  │   AI加速单元        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Linux V4L2 子系统

```
┌─────────────────────────────────────────────────────────────┐
│                     用户空间 (User Space)                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  Camera2   │  │  GStreamer  │  │    V4L2 Utils      │ │
│  │  API App   │  │   Pipeline  │  │   (v4l2-ctl)       │ │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
│         │                │                      │             │
│         └────────────────┼──────────────────────┘             │
│                          ▼                                    │
│               ┌───────────────────────┐                       │
│               │   V4L2 Driver (VIDIOC) │                    │
│               │  ioctl 接口层          │                    │
│               └───────────┬───────────┘                       │
├─────────────────────────────────────────────────────────────┤
│                     内核空间 (Kernel Space)                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐     │
│  │              V4L2 子系统                             │     │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │     │
│  │  │Video   │ │Media   │ │V4L2    │ │Camera  │  │     │
│  │  │Device  │ │Device  │ │Subdev  │ │Sensor  │  │     │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │     │
│  └─────────────────────────────────────────────────────┘     │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐     │
│  │           平台驱动 (Platform Driver)                 │     │
│  │  - msm_camera / qcom_camera (高通)                │     │
│  │  - mediatek_camera (联发科)                         │     │
│  │  - exynos-camera (三星)                            │     │
│  └─────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心实现

### 1. V4L2 核心数据结构

**Video Device**：

```c
// include/uapi/linux/videodev2.h
struct v4l2_capability {
    __u32   driver;      // 驱动名称 "uvc", "msm_vidc"
    __u32   card;        // 设备名称
    __u32   bus_info;   // 总线信息
    __u32   version;    // 驱动版本
    __u32   capabilities; // V4L2_CAP_* 标志
    __u32   device_caps;
};

// 关键 capability
#define V4L2_CAP_VIDEO_CAPTURE       0x00000001  // 视频捕获
#define V4L2_CAP_VIDEO_OUTPUT        0x00000002  // 视频输出
#define V4L2_CAP_STREAMING           0x10000000  // 流式传输
#define V4L2_CAP_DEVICE_CAPS        0x80000000  // 设备能力
```

**Buffer 管理**：

```c
// 帧缓冲描述
struct v4l2_buffer {
    __u32           index;      // buffer 序号
    __u32           type;       // V4L2_BUF_TYPE_VIDEO_CAPTURE
    __u32           bytesused;  // 使用字节数
    __u32           flags;      // V4L2_BUF_FLAG_*
    __u32           field;      // 场序
    struct timeval  timestamp;  // 时间戳
    __u32           sequence;   // 帧序号
    __u32           memory;     // V4L2_MEMORY_* (MMAP/USERPTR/DMABUF)
    union {
        __u32           offset;   // MMAP 偏移
        __u64           userptr;  // 用户指针
        __s32           fd;      // DMABUF fd
    } m;
    __u32           length;    // buffer 长度
    __u32           reserved2;
};

// DMABUF 共享关键
#define V4L2_MEMORY_MMAP      0  // 内核映射
#define V4L2_MEMORY_USERPTR    1  // 用户指针
#define V4L2_MEMORY_DMABUF    4  // DMA-BUF fd (零拷贝!)
```

### 2. Camera HAL 接口

**HAL3 核心结构**：

```c
// hardware/interfaces/camera/common/1.0/types.hal
interface ICameraDevice {
    // 获取设备信息
    getResourceCost() generates (ResourceCost resourceCost);
    
    // 获取静态信息
    getStreamConfiguration() generates (StreamConfiguration supportedConfigurations);
    
    // 构建处理块
    constructDefaultStreamConfiguration() generates (StreamConfiguration configuration);
    
    // 打开设备
    open(in hardware.ICameraClient callback) generates (Status status);
    
    // 捕获请求
    processCaptureRequest(in CaptureRequest request) generates (Status status);
};

// 捕获请求
struct CaptureRequest {
    vec<StreamBuffer> outputBuffers;
    vec<StreamBuffer> inputBuffer;
    CaptureSettings settings;
    uint32_t frameNumber;
};
```

### 3. ISP 流水线

**典型 ISP 处理流程**：

```
Sensor RAW数据 (Bayer)
        │
        ▼
┌─────────────────────────────────────────────────────────────┐
│                   ISP Pipeline                               │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │ BLC     │  │Demosaic │  │  AWB    │  │  AEC    │       │
│  │黑电平补偿│  │去马赛克 │  │白平衡   │  │自动曝光  │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐       │
│  │  CCNR   │  │ 锐化   │  │ 色彩校正│  │ Gamma   │       │
│  │降噪     │  │Sharpen  │  │CCM      │  │ 曲线    │       │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘       │
└─────────────────────────────────────────────────────────────┘
        │
        ▼
    YUV/RGB 输出
```

### 4. DMA-BUF 零拷贝

**Camera → Display 零拷贝**：

```c
// 1. Camera 导出 DMA-BUF
int sensor_get_dmabuf(struct sensor_buffer *buf, int *dma_fd) {
    struct dma_buf *dmabuf = buf->dmabuf;
    *dma_fd = dma_buf_fd(dmabuf);  // 获取 fd
    return 0;
}

// 2. 通过 V4L2 导出
struct v4l2_exportbuffer expbuf = {0};
expbuf.type = V4L2_BUF_TYPE_VIDEO_CAPTURE_MPLANE;
expbuf.index = buffer_index;
expbuf.flags = O_RDONLY;
ioctl(v4l2_fd, VIDIOC_EXPBUF, &expbuf);  // 获取 dmabuf fd

// 3. DRM import 显示
struct drm_prime_handle_args args = {
    .fd = expbuf.fd,
    .flags = DRM_CLOEXEC
};
ioctl(drm_fd, DRM_IOCTL_PRIME_FD_TO_HANDLE, &args);

// 4. 直接显示，无需 CPU 拷贝
drmModeSetCrtc(drm_fd, crtc_id, fb_id, ...);
```

---

## 功耗优化 (单独章节)

### 1. 系统级功耗架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Camera 功耗模型                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  总功耗 = P_sensor + P_isp + P_ddr + P_bus                 │
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │   Sensor   │  │    ISP     │  │    DDR     │       │
│  │  图像采集  │  │  算法处理  │  │  数据带宽  │       │
│  │ 100-500mW │  │ 200-800mW  │  │ 50-200mW   │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2. 分场景功耗策略

| 场景 | 帧率 | 分辨率 | ISP频率 | 典型功耗 |
|------|------|--------|---------|----------|
| **预览** | 30fps | 1080p | 中 | 300-500mW |
| **拍照** | 单帧 | 108MP | 峰值 | 800-1500mW |
| **4K视频** | 30fps | 4K | 高 | 1.5-2.5W |
| **1080P视频** | 30fps | 1080p | 中 | 800-1200mW |
| **慢动作** | 240fps | 720p | 超高 | 2-3W |
| **夜景模式** | 15fps | 多帧合成 | 中 | 1-1.5W |

### 3. 动态功耗控制

**帧率动态调整**：

```c
// 根据场景调整帧率
static int adjust_fps_by_scene(struct camera_ctx *ctx) {
    struct fps_policy *policy = &ctx->fps_policy;
    
    // 检测运动场景 → 提高帧率
    if (ctx->motion_detected) {
        policy->target_fps = 60;
        policy->isp_freq = ISP_FREQ_HIGH;
    }
    // 静止场景 → 降低帧率省电
    else if (ctx->static_frames > 30) {
        policy->target_fps = 24;
        policy->isp_freq = ISP_FREQ_LOW;
    }
    
    return 0;
}
```

**传感器休眠策略**：

```c
// 预览时降低功耗
static int sensor_set_low_power_mode(struct sensor_dev *sensor, bool enable) {
    if (enable) {
        // 降低工作时钟
        sensor->set_clk(sensor, SENSOR_CLK_24MHZ);
        // 降低帧率
        sensor->set_fps(sensor, 15);
        // 进入低功耗模式
        sensor->set_mode(sensor, SENSOR_MODE_LOW_POWER);
    } else {
        // 恢复正常
        sensor->set_clk(sensor, SENSOR_CLK_24MHZ);
        sensor->set_fps(sensor, 30);
        sensor->set_mode(sensor, SENSOR_MODE_NORMAL);
    }
}
```

### 4. AI 场景检测优化

**按需启动 NPU**：

```c
// 仅在需要时启动AI检测
static bool should_enable_ai_scene(struct camera_ctx *ctx) {
    // 用户手动开启AI场景
    if (ctx->user_ai_enabled)
        return true;
    
    // 检测到特定场景类型
    if (ctx->current_scene == SCENE_NIGHT ||
        ctx->current_scene == SCENE_PORTRAIT ||
        ctx->current_scene == SCENE_FOOD)
        return true;
    
    return false;
}
```

### 5. DDR 带宽优化

**数据压缩传输**：

```c
// 使用 ISP 硬件压缩减少 DDR 带宽
static int isp_enable_compression(struct isp_dev *isp, bool enable) {
    if (enable) {
        // 启用 JPEG 硬件压缩
        isp->config.compress.enable = true;
        isp->config.compress.format = COMPRESS_JPEG;
        // 或使用 MIPI CSI-2 压缩
        isp->config.mipi_compress = MIPI_CCOMPRESS_10BIT;
    }
    return 0;
}
```

### 6. 协同调度

**ISP/VPU/NPU 协同**：

```
┌─────────────────────────────────────────────────────────────┐
│              计算摄影任务分配                                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐       │
│  │    ISP    │    │    VPU    │    │    NPU    │       │
│  │ 基础图像  │───▶│ 视频编码   │───▶│ AI场景检测 │       │
│  │ 处理(DSP) │    │ (H.265)   │    │ (AI Model)│       │
│  └────────────┘    └────────────┘    └────────────┘       │
│       │                                       │             │
│       └────────────────┬──────────────────────┘             │
│                        ▼                                      │
│              ┌─────────────────────┐                          │
│              │   统一调度器        │                          │
│              │ 任务依赖 + 功耗预算  │                          │
│              └─────────────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

### 7. 关键技术点总结

| 优化点 | 方法 | 功耗降低 |
|--------|------|----------|
| **帧率** | 动态调整 FPS | 20-40% |
| **分辨率** | 预览/录制不同分辨率 | 30-50% |
| **传感器** | 低功耗模式 | 50-70% |
| **DDR带宽** | 硬件压缩 | 20-30% |
| **AI按需** | 场景检测后启用 | 15-25% |
| **协同调度** | 任务合并 | 10-20% |

---

## 学习资源

### 核心仓库

| 项目 | Stars | 说明 |
|------|-------|------|
| [torvalds/linux](https://github.com/torvalds/linux) | ⭐222K | V4L2 子系统 |
| [msm_media_driver](https://github.com/andersson/msm_media_driver) | ⭐1.2K | 高通视频驱动 |
| [mediatek_camera](https://github.com/mediatek-labs/blocklyduino-for-linkit) | - | 联发科 Camera |

### 文档

- [Linux Media Subsystem](https://www.kernel.org/doc/html/latest/media/)
- [Android Camera](https://source.android.com/docs/core/camera)
- [V4L2 Spec](https://www.kernel.org/doc/html/latest/userspace-api/media/v4l.html)

### 调试工具

| 工具 | 用途 |
|------|------|
| `v4l2-ctl` | V4L2 设备控制 |
| `v4l2-compliance` | V4L2 合规性测试 |
| `systrace` | 系统级跟踪 |
| `camx` | 高通 Camera 框架 |

---

## 🔥 核心技术价值

> **V4L2 子系统** - `drivers/media/`
> - Video/Media/Subdev 驱动模型
> - DMABUF 零拷贝机制
> - **潜在平台价值**: Camera驱动定制/多摄协同

> **Camera HAL3** - `hardware/interfaces/camera/`
> - 统一 HAL 接口标准
> - 计算摄影流水线
> - **潜在平台价值**: 相机效果定制/算法优化

> **ISP Pipeline** - 图像信号处理
> - Demosaic/AWB/AEC/降噪
> - 硬件流水线低延迟
> - **潜在平台价值**: 夜景模式/HDR/美颜

> **功耗优化**
> - 帧率/分辨率动态调整
> - 传感器低功耗模式
> - ISP/VPU/NPU 协同调度
> - **潜在平台价值**: 续航提升/发热控制
