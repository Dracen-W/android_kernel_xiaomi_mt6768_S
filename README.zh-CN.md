# 小米 Redmi Note 9 / 10X 4G (merlin) 内核

**SoC：** 联发科 MT6768 (Helio G85) · **内存：** 4 GB · **代号：** merlin / merlinx

[English README → README.md](README.md)

---

## 基线

基于 [mt6768-dev/android_kernel_xiaomi_mt6768](https://github.com/mt6768-dev/android_kernel_xiaomi_mt6768)。
本分支加入了完整的 **DroidSpaces** 支持，移植了 **r49p1** GPU DDK，并
backport 了一批安全、存储与内存管理特性（详见
[内核 DDK 驱动适配到 r49p1](#内核-ddk-驱动适配到-r49p1)）。

## Backport 特性

### 安全 backport

| 特性 | 作用 |
|------|------|
| **SCS**（影子调用栈） | arm64 影子调用栈，防御 ROP/JOP 攻击 |
| **LIST_HARDENED** | 链表完整性校验，缓解 UAF / 溢出 |
| **BPF 加固** | eBPF 指针检查与 helper 校验 |
| **CI / clang 构建** | clang-14 + LTO 构建流程 |

### 存储 backport

| 特性 | 作用 |
|------|------|
| **zram** | 上游 writeback 修复与改进 |
| **f2fs** | 周期性检查点间隔 60s → 300s（减少 eMMC 写入停顿） |
| **EROFS** | system/vendor 只读压缩文件系统（更省空间、开机更快） |

### 内存管理 backport

| 特性 | 作用 |
|------|------|
| **MADV_COLD / MADV_PAGEOUT** | 主动内存回收提示（供 lmkd 使用） |
| **MGLRU**（多代 LRU） | 源自 Google ACK android13-5.10 的多代页面回收 —— 冷热页识别更精准，4 GB 机型后台保活 +10~20%，回收 CPU 开销 -20~40% |
| **process_madvise** | 跨进程 madvise 系统调用 —— lmkd 通过 pidfd 对后台 App 主动回收 |

## 内核 DDK 驱动适配到 r49p1

本内核中的 r49p1 **Bifrost** 驱动源码来自 **ARM**，适配工作参考了内核中
原有的 **r32** 源码完成。适配后的内核态 DDK **向下兼容原厂 r32 vendor
blob** —— 内核可单独刷入，无需改动 vendor 分区。

### Vulkan 1.1 → 1.3.278

升级 **Vulkan 从 1.1 到 1.3.278** 请使用以下仓库：

> https://github.com/Dracen-W/r49p1_GPUDriverUpgrade_for_redmi_merlinx_mt6768_vendorS

该仓库提供 **vendor 侧（用户态）** GPU 驱动 blob，形式为 **Magisk 模块**。
两者**不需要一起刷入**，但**顺序很重要**：

1. **先刷入本内核**（搭配原厂 r32 vendor blob 即可直接使用）。
2. **再**通过 **Magisk** 安装驱动 blob 模块，挂载升级后的用户态驱动。

> **⚠️ Bootloop 警告**：**先装 r49p1 驱动模块、后刷本内核**会直接
> **bootloop**。务必先刷内核，再装 Magisk 模块。

Vulkan 1.3.278 相对 1.1 的主要新特性：

- Dynamic Rendering（免 Render Pass / framebuffer 对象）
- Synchronization2（增强的同步原语）
- Maintenance5 等维护性扩展
- 更好的多线程命令录制性能

> **⚠️ 稳定性说明**：当前驱动处于 **"未测到已知 bug"** 状态 —— 这**不等于
> "没有 bug"**。未发现问题仅表示在现有测试范围内未发现异常，不代表所有场景
> 均无问题。刷入前请做好备份，风险自负。

## 构建

```bash
make O=out ARCH=arm64 CC=clang-14 LD=ld.lld-14 LLVM=1 LLVM_IAS=1 \
    vendor/mt6768_defconfig
```

CI 构建与测试通过本仓库的 GitHub Actions 运行。

## 参见

- [English README](README.md)
- [vendor 侧 GPU 驱动升级（Vulkan 1.3.278）](https://github.com/Dracen-W/r49p1_GPUDriverUpgrade_for_redmi_merlinx_mt6768_vendorS/blob/main/README.zh-CN.md)
