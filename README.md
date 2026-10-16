# Linux Kernel for Xiaomi Redmi Note 9 / Redmi 10X 4G (merlin)

**SoC:** MediaTek MT6768 (Helio G85) · **RAM:** 4 GB · **Codename:** merlin / merlinx

[中文说明 → README.zh-CN.md](README.zh-CN.md)

> The pristine upstream Linux kernel documentation is kept unchanged in
> [`README`](README) (see also the [upstream Linux kernel README](https://github.com/torvalds/linux/blob/master/README)).

---

## Base

Based on Android Common Kernel (ACK) **android-4.19-stable r49p1** (Xiaomi S branch).
This branch backports a set of security, storage, and memory-management features,
and adapts the in-kernel GPU DDK drivers to **r49p1**.

## Backported features

### Batch 1 — Security hardening

| Feature | Purpose |
|---------|---------|
| **SCS** (Shadow Call Stack) | arm64 shadow call stack, mitigates ROP/JOP attacks |
| **LIST_HARDENED** | Link-list integrity checks, mitigates UAF / overflow |
| **BPF hardening** | eBPF pointer checks and helper validation |
| **CI / clang build** | clang-14 + LTO build pipeline |

### Batch 2 — Storage

| Feature | Purpose |
|---------|---------|
| **zram** | Upstream writeback fixes and improvements |
| **f2fs** | Periodic checkpoint interval 60 s → 300 s (fewer eMMC write stalls) |
| **EROFS** | Read-only compressed filesystem for system/vendor (smaller, faster boot) |

### Batch 3 — Memory management

| Feature | Purpose |
|---------|---------|
| **MADV_COLD / MADV_PAGEOUT** | Proactive memory-reclaim hints (used by lmkd) |
| **MGLRU** (Multi-Gen LRU) | Multi-generation page reclaim from Google ACK android13-5.10 — more accurate hot/cold classification, ~10–20 % better background-app retention and ~20–40 % lower reclaim CPU overhead on 4 GB devices |
| **process_madvise** | Cross-process madvise syscall — lmkd reclaims memory from background apps via pidfd |

## GPU DDK adaptation to r49p1

The in-kernel GPU / multimedia DDK drivers are adapted to **r49p1** to pair with the
userspace GPU driver.

### Vulkan 1.1 → 1.3.278

Pair this kernel with the following repository to upgrade **Vulkan from 1.1 to 1.3.278**:

> https://github.com/Dracen-W/r49p1_GPUDriverUpgrade_for_redmi_merlinx_mt6768_vendorS

That repository provides the **vendor-side (userspace)** GPU driver upgrade; this
repository provides the matching **kernel-side DDK interfaces**. Both must be
flashed together.

Highlights of Vulkan 1.3.278 vs 1.1:

- Dynamic Rendering (no render-pass / framebuffer objects)
- Synchronization2 (enhanced sync primitives)
- Maintenance5 and other maintenance extensions
- Better multi-threaded command-buffer recording

> **⚠️ Stability disclaimer:** The driver is currently in a
> **"no known bugs found"** state — this does **not** mean it is **bug-free**.
> The absence of observed issues only means none were found within the scope of
> current testing. Flash at your own risk and keep a backup.

## Building

```bash
make O=out ARCH=arm64 CC=clang-14 LD=ld.lld-14 LLVM=1 LLVM_IAS=1 \
    vendor/mt6768_defconfig
```

CI builds and tests are run via GitHub Actions on this repository.

## See also

- [中文说明 (Chinese README)](README.zh-CN.md)
- [Vendor-side GPU driver upgrade (Vulkan 1.3.278)](https://github.com/Dracen-W/r49p1_GPUDriverUpgrade_for_redmi_merlinx_mt6768_vendorS)
