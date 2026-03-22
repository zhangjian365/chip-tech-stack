# 图形技术栈 (Graphics Stack)

## 目录
- [为什么有价值](#为什么有价值-痛点分析)
- [架构设计](#架构设计)
- [核心实现](#核心实现)

---

## 为什么有价值 - 痛点分析

### 1. 移动端渲染瓶颈

**痛点**：
- 游戏/AR 应用对帧率要求高 (60fps+)
- GPU 算力受限，功耗敏感
- 多任务场景下渲染稳定性

**价值**：掌握图形栈才能进行针对性的性能优化

### 2. 异构计算趋势

**痛点**：
- CPU/GPU/NPU 协同计算
- 统一内存 (UMA) 带来的数据共享
- 任务如何在各计算单元间分配

**价值**：图形栈是异构计算的典型场景

### 3. 定制化需求

**痛点**：
- 厂商需要差异化渲染效果
- DRM 保护内容需要安全渲染
- AI 增强渲染 (DLSS, FSR)

**价值**：深入理解才能做深度定制

---

## 架构设计

### Linux 图形栈整体架构

```
┌─────────────────────────────────────────────────────────────┐
│                     用户空间 (User Space)                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │   游戏/APP   │  │   Wayland   │  │    Vulkan/GLES      │  │
│  │  (Surface)  │  │  Compositor │  │   (Driver ICD)      │  │
│  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘  │
│         │                │                      │             │
│         └────────────────┼──────────────────────┘             │
│                          ▼                                    │
│               ┌───────────────────────┐                       │
│               │   Mesa / Driver ICD   │                       │
│               │  (DRI, VKMS, KMD)     │                       │
│               └───────────┬───────────┘                       │
│                          ▼                                    │
├─────────────────────────────────────────────────────────────┤
│                     内核空间 (Kernel Space)                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐     │
│  │              DRM/KMS 子系统                           │     │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │     │
│  │  │Display  │ │ CRTC    │ │ Encoder │ │ Connector│  │     │
│  │  │Driver   │ │         │ │         │ │          │  │     │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │     │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────────────────┐  │     │
│  │  │GEM/     │ │ KMS     │ │    Render Node      │  │     │
│  │  │PRIME    │ │         │ │   (dri/renderD*)    │  │     │
│  │  └─────────┘ └─────────┘ └─────────────────────┘  │     │
│  └─────────────────────────────────────────────────────┘     │
│                          ▼                                    │
│  ┌─────────────────────────────────────────────────────┐     │
│  │              Mali GPU Driver (mali_kbase)           │     │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐  │     │
│  │  │Job      │ │ MMU     │ │ Memory  │ │ Power   │  │     │
│  │  │Scheduler│ │ Table   │ │ Manager │ │ Manager │  │     │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘  │     │
│  └─────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

### 核心组件关系

| 组件 | 层次 | 作用 |
|------|------|------|
| **Wayland** | 协议/合成器 | 替代 X11 的显示服务器协议 |
| **Mesa** | 用户态驱动 | 开源 GL/Vulkan 实现 |
| **DRM** | 内核子系统 | 图形资源管理、显示控制 |
| **KMS** | DRM 子模块 | 显示模式设置、CRTC/Plane 管理 |
| **VKMS** | 虚拟 KMS | 纯软件实现的 KMS (测试用) |
| **mali_kbase** | 内核驱动 | ARM Mali GPU 设备驱动 |

---

## 核心实现

### 1. DRM 子系统

**核心数据结构**：

```c
// drivers/gpu/drm/drm_device.c
struct drm_device {
    struct device *dev;
    struct drm_driver *driver;
    struct drm_master *master;
    struct idr *object_name_idr;
    
    // KMS 核心对象
    struct list_head mode_config.crtc_list;
    struct list_head mode_config.connector_list;
    struct list_head mode_config.encoder_list;
    struct list_head mode_config.plane_list;
    
    // 显存管理
    struct drm_gem_object_manager *gem;
};

// drm_mode.h
struct drm_crtc {
    struct drm_base_object base;
    struct drm_plane *primary_plane;
    struct drm_plane *cursor_plane;
    
    // 扫描输出
    struct drm_display_mode *mode;
    uint32_t *gamma_store;
    
    // Framebuffer
    struct drm_framebuffer *fb;
};
```

**关键流程 - 显示模式设置**：

```
用户空间 (weston-compose)
       │
       ▼
drmModeSetCrtc (ioctl)
       │
       ▼
drm_crtc_set_config()
       │
       ├─▶ drm_plane_set_fb()   // 设置 Plane Framebuffer
       │
       └─▶ drm_bridge_chain_enable() // 启用显示桥接
```

### 2. Mali GPU 驱动

**驱动架构** (`drivers/gpu/arm/mali_bifrost`)：

```c
// mali_kbase_defs.h
struct kbase_device {
    struct device *dev;
    
    // GPU 硬件信息
    const struct kbase_gpu_props *gpu_props;
    
    // 作业调度
    struct kbase_job_scheduler *js_scheduler;
    
    // 内存管理
    struct kbase_mmu_table *mmu;
    
    // 电源管理
    struct kbase_pm_device *pm;
};

// mali_kbase_jd.c - 作业调度
struct kbase_jd_atom {
    u64 job_chain_addr;      // GPU 命令链地址
    enum kbase_jd_atom_state state;
    struct kbase_context *kctx;
    
    // 依赖关系
    struct list_head dep_list[2];
};
```

**渲染管线**：

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
         │
    ┌────┴────┐
    ▼         ▼
 Job     Memory
Scheduler Manager
    │         │
    ▼         ▼
 GPU HW   Page Table
```

### 3. GEM/PRIME 显存管理

**核心机制**：

```c
// drm/gem.c
struct drm_gem_object {
    struct kref refcount;
    size_t size;
    struct file *filp;
    
    // 物理页管理
    struct sg_table *sgt;
    void *mapping;
    
    // 导出/共享
    uint32_t handle;
};

// PRIME - 跨设备共享
struct drm_prime_member {
    struct drm_device *peer;
    uint32_t fd;
    struct dma_buf *dma_buf;
};
```

**Buffer 共享流程**：
```
Camera (V4L2)              Display (DRM)
     │                           │
     ▼                           │
dmabuf export ──────────────────►│
     │                           │
     │                     dmabuf import
     │                           │
     ▼                           ▼
  Camera Buffer             Framebuffer
```

### 4. Wayland 合成器

**架构** (以 Weston 为例)：

```
┌────────────────────────────────────┐
│         Clients (游戏/应用)         │
│   ┌────┐  ┌────┐  ┌────┐          │
│   │wl_ │  │wl_ │  │wl_ │          │
│   │seat│  │dma-│  │sur-│          │
│   │    │  │buf │  │face│          │
│   └────┘  └────┘  └────┘          │
└──────────────┬─────────────────────┘
               │ Wayland Protocol
               ▼
┌────────────────────────────────────┐
│        Weston Compositor            │
│  ┌────────────────────────────────┐ │
│  │ weston_output (显示输出)        │ │
│  │ weston_view (视图/平面)         │ │
│  │ weston_layer (层级)             │ │
│  └────────────────────────────────┘ │
│  ┌────────────────────────────────┐ │
│  │ gl-renderer (GLES 合成)        │ │
│  │ pixman-renderer (软件渲染)      │ │
│  └────────────────────────────────┘ │
└──────────────┬─────────────────────┘
               │ DRM/KMS
               ▼
┌────────────────────────────────────┐
│           DRM Device                 │
└────────────────────────────────────┘
```

### 5. Vulkan 驱动接口

**驱动加载**：

```c
// vulkan/util/vulkan_loader.c
typedef struct VkAllocationCallbacks {
    void* pUserData;
    PFN_vkAllocationFunction pfnAllocation;
    PFN_vkReallocationFunction pfnReallocation;
    PFN_vkFreeFunction pfnFree;
    PFN_vkInternalAllocationNotification pfnInternalAllocation;
    PFN_vkInternalFreeNotification pfnInternalFree;
} VkAllocationCallbacks;

// 驱动 ICD 加载
vkEnumerateInstanceExtensionProperties()
    │
    ▼
dlopen("libvulkan_mali.so")  // 厂商驱动
    │
    ▼
vkCreateInstance() → vkGetPhysicalDeviceQueueFamilyProperties()
```

---

## 关键技术点

### 零拷贝渲染

**原理**：通过 DMA-BUF 共享显存，避免 CPU 拷贝

```c
// 典型场景：Camera → GPU → Display
// 1. Camera 导出 dmabuf
int dma_buf_fd = export_dmabuf(camera_buffer);

// 2. DRM import dmabuf
struct drm_prime_handle_args args = {
    .fd = dma_buf_fd,
    .flags = DRM_CLOEXEC
};
drmIoctl(drm_fd, DRM_IOCTL_PRIME_HANDLE_TO_FD, &args);

// 3. 绑定到 Framebuffer
drmModeAddFB2(drm_fd, width, height, format, 
              handles, pitches, offsets, &fb_id, 0);

// 4. 显示输出
drmModeSetCrtc(drm_fd, crtc_id, fb_id, 0, 0, 
               connector_ids, 1, &mode);
```

### 虚拟显示 (VKMS)

**用途**：无硬件环境下的图形栈开发/测试

```c
// drivers/gpu/drm/vkms/vkms_drv.c
static struct drm_driver vkms_driver = {
    .driver_features = DRIVER_MODESET | DRIVER_ATOMIC,
    .name = "vkms",
    .desc = "Virtual Kernel Mode Setting",
    
    .lastclose = vkms_lastclose,
    .mode_config_init = vkms_mode_config_init,
};

// 纯软件实现的功能
// - CRTC: 虚拟扫描生成
// - Plane: 像素格式转换
// - Connector: 固定分辨率
```

---

## 移动端特殊优化

### 1. Mali Midgard/Bifrost 架构

| 架构 | 关键特性 | 典型设备 |
|------|----------|----------|
| **Midgard** | Tile-based 渲染, 4-wide warp | Mali-T880 |
| **Bifrost** | FBDC 压缩, Exec Mask | Mali-G71 |
| **Valhall** | 128-wide SIMD, V10 | Mali-G77 |
| **Immortalis** | 硬件光追, V12 | Mali-G720 |

### 2. 功耗优化策略

```c
// 内核: devfreq 动态调频
// drivers/gpu/arm/mali/mali_devfreq.c
static int mali_devfreq_target(struct device *dev, 
                               unsigned long *freq,
                               u32 flags) {
    struct mali_device *mali = dev_get_drvdata(dev);
    
    // 根据负载动态调整频率
    if (mali->utilization > 80)
        *freq = mali->max_freq;
    else if (mali->utilization < 20)
        *freq = mali->min_freq;
        
    return 0;
}
```

---

## 学习资源

### 官方文档
- [Linux DRM 文档](https://www.kernel.org/doc/html/latest/gpu/index.html)
- [Mesa 3D 文档](https://docs.mesa3d.org/)
- [Wayland 协议](https://wayland.freedesktop.org/docs/html/)

### 核心仓库

| 项目 | Stars | 说明 |
|------|-------|------|
| [torvalds/linux](https://github.com/torvalds/linux) | ⭐222K | 内核 DRM/KMS |
| [Mesa3D/mesa](https://github.com/Mesa3D/mesa) | ⭐5.3K | GL/Vulkan 实现 |
| [wayland-project/weston](https://github.com/wayland-project/weston) | ⭐1.5K | Wayland 合成器参考实现 |
| [MaliGraphics/mali](https://github.com/MaliGraphics/mali-kernel) | - | Mali 驱动源码分析 |

### 调试工具

| 工具 | 用途 |
|------|------|
| `modetest` | DRM/KMS 测试 |
| `glxinfo` | OpenGL 信息查询 |
| `vulkaninfo` | Vulkan 信息查询 |
| `weston-info` | Wayland 客户端调试 |
| `drm_debug` | DRM 内核调试 |

---

## 🔥 核心技术价值

> **DRM/KMS 显示管线** - `drivers/gpu/drm/`
> - 架构核心: CRTC → Encoder → Connector 驱动模型
> - 原子更新: drm_atomic_commit 实现无闪烁切换
> - **潜在平台价值**: 车载显示/多屏异显/Always-On

> **Mali GPU 驱动** - `drivers/gpu/arm/mali_bifrost/`
> - Job Scheduler: GPU 作业调度与依赖管理
> - Memory Manager: 页表管理与 TLB 刷新
> - **潜在平台价值**: 游戏渲染/AI 推理加速/视频编解码

> **Wayland 合成器** - `compositor/`
> - 平面合成: 多个 Surface 按 Z-order 合成
> - 客户端缓冲: dmabuf 共享机制
> - **潜在平台价值**: 车机 HMI/桌面环境/AR 眼镜

> **零拷贝显示** - DMA-BUF
> - 跨设备缓冲共享: Camera → GPU → Display
> - 无 CPU 参与的数据搬运
> - **潜在平台价值**: 视频预览/屏幕录制/多路编解码
