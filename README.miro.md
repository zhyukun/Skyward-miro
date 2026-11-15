# Skyward-miro — REDMI K80 Pro 适配

本仓库是 [kirklin/skyward](https://github.com/kirklin/skyward) 的个人 fork，仅用于 **miro** 机型适配。上游项目与原作者 Kirk Lin 的版权、`LICENSE` 均原样保留；适配部分由 **zhyukun** 完成。

- 适配分支：`miro`；`main` 跟踪上游 `kirklin/skyward`。
- 上游通用说明见 [README.md](README.md) / [README.zh-CN.md](README.zh-CN.md)，本文件只记录本机适配状态与已知问题。

## 机型

| 项 | 值 |
|---|---|
| 设备 | REDMI K80 Pro / POCO F7 Ultra，代号 `miro`，国行 24122RKC7C |
| SoC | 骁龙 8 Elite SM8750（平台 sun，soc_id 618） |
| 系统基底 | Android 17 / SDK 37，HyperOS 4.0.0.8，16KB 页，arm64-v8a |
| 分区方案 | A/B，独立 recovery 分区；当前活动槽 `_a` |

## 当前状态（v9，2026-11-15 编译）

已在真机验证：

- **触摸**：锁屏 / 息屏唤醒后仍可用（触摸设置由后台线程每秒定时重写），备份与恢复可用。
- **data 解密**：冷启动输入锁屏密码可一次解密进入。
- **WLAN**：peach_v2 主机驱动，`wlan0` 在进入 WLAN 页时按需 probe。
- **其他**：槽位读取、persist 挂载、BootControl 服务、MTP / ADB 可用。

## 已知问题与未完成

1. **data 解密稳定性未坐实**
   - 仅冷启动一次通过，未做多轮对照；偶发失败的精确原因未落盘。
   - 服务就绪竞争的根因尚未确认；strongbox 实例始终未拉起。
   - 错误码 `-28` 的确切枚举名未核实；“立即解密 vs 预热后解密”的对照未做。
   - 整机 s2idle 是否叠加触摸因素，尚未排除。
2. **WLAN 非致命告警**：`hdd_config_data_stall_param` fail、bmiss sysfs 节点创建失败、6GHz disabled、MLO MLD ID 0 exists；`wlan0` 需进入 WLAN 页才 probe。
3. **振动马达**：沿用 17 Ultra 的谐振频率与强度参数，尚未在本机马达上实测。
4. **尚未纳入的上游改动**：open PR #33（crypto 遍历所有 protector）、#29（pack 纳入 stock ramdisk 缺文件）、#60（WLAN firmware alternatives）、#48（myron A17 PIN 解密）、#43（socrates 解密兼容）。
5. **系统版本**：当前基于 HyperOS 4.0.0.8 基底；若升级 4.0.3.0，需重新 dump recovery 基底并重打包。
6. **更新机制**：仅在手动进入更新页且在线时向上游查询一次；校验 asset 机型名与 SHA-256，**无独立 GPG / 签名验签**。

## 构建

- 环境：WSL2 Ubuntu-24.04，Rust 1.100.0，Android NDK r29，cargo-ndk 4.1.2，交叉编译 `--jobs 4`。
- 测试：`cargo test --workspace --locked --jobs 4`。
- 打包使用上游 `skyward-pack`，指定 stock recovery 基底、手机端二进制、`system-init`、`devices/miro.toml` 与字体，输出 recovery 镜像。

## 刷入与回退

- 仅刷 recovery 分区：`fastboot flash recovery_a <image.img>`，随后 `fastboot reboot recovery`。
- **不触碰** boot / init_boot / vendor_boot / system / vendor / data；操作中请勿点击任何 Wipe、格式化或恢复出厂。
- 回退：fastboot 刷回原厂 recovery 备份或上一版镜像。

## 免责

截至 2026-11-15 编译时，上游 Skyward 尚未发布适配本机（REDMI K80 Pro）HyperOS 4.0 的恢复模式映像包。若检测到更新，请务必确认待升级包支持当前机型与 Android 版本。因刷入错误映像造成的数据丢失、设备损坏等一切后果，适配者 zhyukun 概不承担责任。

## 许可

上游 GPL-3.0-or-later；原作者 Kirk Lin 版权与 `LICENSE_ORIGINAL` 原文保留。本 fork 的适配改动同样以 GPL-3.0-or-later 发布。
