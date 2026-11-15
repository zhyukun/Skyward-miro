![Skyward Recovery](assets/hero.jpg)

# Skyward V0.3.1 For Miro

[English](README.md) | 简体中文

> **miro 适配分支** —— [kirklin/skyward](https://github.com/kirklin/skyward) 的个人 fork，由 zhyukun 完成 REDMI K80 Pro（`miro`）适配。上游项目、原作者 Kirk Lin 的版权与 `LICENSE` 均原样保留，本 fork 只在其上追加适配改动。适配内容位于 `miro` 分支；`main` 保留上游基线不变。

## 机型

| 项 | 值 |
| --- | --- |
| 设备 | REDMI K80 Pro / POCO F7 Ultra，代号 `miro`，国行 24122RKC7C |
| SoC | 骁龙 8 Elite SM8750（平台 sun，soc_id 618） |
| 系统基底 | Android 17 / SDK 37，HyperOS 4.0.0.8，16KB 页，arm64-v8a |
| 分区方案 | A/B，独立 recovery 分区；当前活动槽 `_a` |
| 编译 | 2026-11-15，发布名 **Skyward V0.3.1 For Miro** |

> [!WARNING]
> ### 适用范围 · 刷入前必读
> 本恢复环境映像包目前仅在运行 **HyperOS 4.0.0.8（Android 17）** 的 REDMI K80 Pro（`miro`）上测试，其他设备不建议刷入。同型号搭载 **HyperOS 3.X（Android 16）** 的手机也**不建议**刷入——Android 版本不同。

## 已在真机验证

- **触摸**：锁屏 / 息屏唤醒后仍可用（触摸设置由后台线程每秒定时重写），备份与恢复可用。
- **data 解密**：冷启动输入锁屏密码可一次解密进入。
- **WLAN**：peach_v2 主机驱动，`wlan0` 进入 WLAN 页时按需 probe。
- **其他**：槽位读取、persist 挂载、BootControl 服务、MTP / ADB。

## 下载

镜像：[skyward-0.3.1-miro-a17-zhyukun.img](https://github.com/zhyukun/Skyward-miro/releases/download/v1/skyward-0.3.1-miro-a17-zhyukun.img) · [Release 页面](https://github.com/zhyukun/Skyward-miro/releases/tag/v1)

```text
file    skyward-0.3.1-miro-a17-zhyukun.img
size    104,857,600 字节（100 MiB）
SHA-256 9e7110209ba45a36897b0560f706300674b8cb4a598de47ed2a505a292a0fd18
```

## 刷入与回退

仅刷 recovery 分区：

```sh
fastboot devices
fastboot flash recovery_a skyward-0.3.1-miro-a17-zhyukun.img
fastboot reboot recovery
```

- 需要已解锁的 bootloader。
- 回退：fastboot 刷回原厂 recovery 备份或上一版镜像。

## 已知问题与未完成

1. **data 解密稳定性尚未坐实**
   - 仅冷启动一次通过，未做多轮对照；偶发失败的精确原因未落盘。
   - 服务就绪竞争的根因尚未确认；strongbox 实例始终未拉起。
   - 错误码 `-28` 的确切枚举名未核实；“立即解密 vs 预热后解密”的对照未做。
   - 整机 s2idle 是否叠加触摸因素，尚未排除。
2. **WLAN 非致命告警**：`hdd_config_data_stall_param` fail、bmiss sysfs 节点创建失败、6GHz disabled、MLO MLD ID 0 exists；`wlan0` 需进入 WLAN 页才 probe。
3. **振动马达**：沿用 17 Ultra 的谐振频率与强度参数，尚未在本机马达上实测。
4. **尚未纳入的上游改动**：open PR #33（crypto 遍历所有 protector）、#29（pack 纳入 stock ramdisk 缺文件）、#60（WLAN firmware alternatives）、#48（myron A17 PIN 解密）、#43（socrates 解密兼容）。
5. **系统版本**：当前基于 HyperOS 4.0.0.8 基底；升级 4.0.3.0 需重新 dump recovery 基底并重打包。
6. **更新机制**：仅在手动进入更新页且在线时向上游查询一次；校验 asset 机型名与 SHA-256，**无独立 GPG / 签名验签**。

## 免责

截至 2026-11-15 编译时，上游 Skyward 尚未发布适配本机（REDMI K80 Pro）HyperOS 4.0 的恢复模式映像包。若检测到更新，请务必确认待升级包支持当前机型与 Android 版本。因刷入错误映像造成的数据丢失、设备损坏等一切后果，适配者 zhyukun 概不承担责任。

---

## 安装 Skyward

Skyward 需要已解锁的 bootloader。从 [Releases](https://github.com/kirklin/skyward/releases) 下载对应机型的 zip，每个文件都标有机型和代号，例如 Xiaomi 17 Ultra (nezha)。使用 `SHA256SUMS` 核对并解压之后，在手机处于 fastboot 模式的时候，把其中的镜像刷入当前运行槽位的 recovery 分区：

```sh
unzip skyward-v0.3.1-nezha-OS4.0.4.0.XPACNXM.zip
fastboot flash recovery skyward-v0.3.1-nezha-OS4.0.4.0.XPACNXM.img
fastboot reboot recovery
```

之后连接 WLAN，就可以在“设置”的“Skyward 更新”页面更新到最新版本，也可以在 Skyward 自己的“安装”页面安装解压出来的新版镜像。打开 **自动重新刷入 Skyward** 之后，系统更新或者刷机包覆盖 recovery 分区的时候，Skyward 会重新刷回这些分区。

Skyward 中的 adb 是 root shell，手机上无需授权：手机处于 recovery 的时候，任何人拿到手机和数据线都能访问其中的存储。

## 双系统的工作方式

Skyward 可以在手机自身分区中的系统旁边安装第二系统，主系统是 HyperOS 或者第三方 ROM 都可以，并在首页的“双系统”中切换。第二系统的根文件系统是 data 分区中的一个文件，文件系统不能移动或删除它，Skyward 直接通过分区读写其中的内容；两个系统的启动镜像保存在另一个这样的文件中。切换系统只改写下次启动槽位的 boot 和 init_boot 分区，每一步都先记录在 /metadata 中，切换过程中断电的话，Skyward 下次启动时会完成这次切换。vendor_boot、dtbo、recovery、vbmeta、固件和 super 都保持不变，第二系统使用主系统的 vendor_boot 和 dtbo 启动。

其他操作写入启动分区之前，Skyward 会先把主系统的启动镜像写回：安装、切换启动槽位、fastboot、恢复出厂设置，以及备份或恢复启动分区。系统更新尚未合并的时候，暂不切换到第二系统；备份 data 分区的时候不包含第二系统。

第二系统的安装包是 zip 文件，和其他安装包一样在“安装”页面安装：

| 文件 | 内容 |
| --- | --- |
| `system.toml` | `id`、`name`、`version` 和 `filesystem`；`init` 是根文件系统中第一个运行的程序，默认为 `/sbin/init`；`minimum` 是根文件系统至少需要的空间，单位为字节 |
| `boot.img` | bootloader 启动的内核 |
| `rootfs.img` | 根文件系统的镜像，格式为 `filesystem` 指定的文件系统 |
| `init_boot.img` | 只有自带第一阶段 ramdisk 的系统需要 |

没有自带 init_boot 镜像的系统，从 Skyward 生成的 init_boot 镜像启动。其中的程序按照主系统第一阶段的方式加载第一阶段的内核模块，包括模块之间的软依赖和内核命令行中给模块的参数，然后在 data 分区中找到根文件系统，挂载之后运行其中的 `init`。启动过程中出现任何错误，这个程序都会进入 Skyward，和主系统启动失败时进入 recovery 一样：它在 misc 中留下原因，把当前槽位 recovery 分区中的 ramdisk 按照 bootloader 启动 recovery 时内核解包的方式解包到自己的 ramdisk 上，然后交给 Skyward 的第一个程序运行，所以无论 bootloader 怎样处理重启，都能进入 Skyward。Skyward 显示原因；用户在“双系统”中切换之前，下次启动的仍是第二系统。启动长时间没有进展时，这个程序改为重启到 recovery；交给第二系统之前手机重启的话，下次启动时它会在 misc 中看到同样的请求，先进入 Skyward。两种方式都无法进入 Skyward 时，手机停留在出错的地方，不会反复启动同一个系统。自带第一阶段 ramdisk 的系统可以运行 Skyward 放在 `/skyward/map` 的程序，它把根文件系统映射为块设备，并输出设备节点的路径。每次切换到第二系统，Skyward 都用自己带的程序写入第二系统的 init_boot 镜像，所以更新 Skyward 以后，第二系统也用新的程序启动。

正在运行的系统也可以请求切换：在 misc 中写入 `boot-recovery` 和 `--switch_system=installed`，或者 `--switch_system=` 加上第二系统的 `id`，然后重启到 recovery。bootloader 仍然启动这个槽位的话（小米 13 Ultra 连着 USB 线时可能这样），Skyward 的程序会在下次启动时在 misc 中看到这个请求，先进入 Skyward。

## 许可证

[GPL-3.0-or-later](LICENSE)
