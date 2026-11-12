# iloader Watch App Group 修复版 · Windows

基于 **AndreySemjonov/iloader v2.3.4-fork.2** 制作的 Windows 命令行版本，补齐 Apple Watch App 的 App Group 配置与描述文件校验，并处理已知 anisette 客户端信息导致的 `-23782` 登录错误。

当前版本：**2.3.4-watchgroup.2**。本项目是社区修改版，表盘功能由 [pookjw/HermesEnabler](https://github.com/pookjw/HermesEnabler) 提供。

## 下载和阅读顺序

- **普通使用者：**在本仓库右侧 **Releases** 下载 `iloader-2.3.4-watchgroup.2-windows-x64.zip`，完整解压，然后阅读 [详细安装教程](INSTALL.zh-CN.md)。
- **遇到报错：**阅读 [踩坑与排查](TROUBLESHOOTING.zh-CN.md)，按错误发生的阶段定位。
- **想了解我们改了什么：**阅读 [修复原理](PATCH-NOTES.zh-CN.md)。
- **想自己编译：**下载 Release 中的 `iloader-2.3.4-watchgroup.2-source.zip`，按 [编译说明](BUILDING.zh-CN.md) 操作。源码包包含 CLI、完整的 vendored isideload 后端和 Cargo.lock。

发布的 Windows ZIP 包含 `iloader-cli.exe`、`Start.cmd`、检查脚本、说明、许可证与校验值。**HermesEnabler IPA 请从原作者发布页下载**，然后放到 EXE 同一目录；本项目提供原文件的 SHA256 供核对。

## 本次实机结果

| 项目 | 本次记录 |
| --- | --- |
| 电脑 | Windows 64 位 |
| 手机 | iPhone 15 Pro / iOS 27.0.1 |
| 手表 | watchOS 27.2（24S118） |
| 原始 IPA | HermesEnabler 1.4.1 |
| 最终程序 | iloader-cli 2.3.4-watchgroup.2 |
| 安装与启用 | 手表 App 可运行，能添加 Hermès 表盘 |
| 完整重启后 | 表盘编辑可以保存，手机 Watch App 可以显示已添加的爱马仕表盘 |

实机结果由使用者在 **2026-11-12** 确认，整个过程没有使用 Mac 或 Xcode。此表记录的是一组成功案例，其他机型、系统、账户与签名有效期需分别确认。

## 最短操作路径

1. 用数据线连接 iPhone，解锁并信任电脑；在 iTunes 中确认能够看到手机详情。
2. 下载、解压本项目的 Windows ZIP，把官方 `HermesEnabler.ipa` 放在同一目录。
3. 保持 iPhone 和手表均已解锁、相互靠近；按系统提示开启开发者模式。
4. 双击 `Start.cmd`，输入自己用于签名的 Apple 账户、密码和验证码。
5. 检查日志中的 `Watch App Group verified in Apple-issued provisioning profile`，等待 `DONE`。
6. 在手表打开 HermesEnabler2，先创建初始 MobileGestalt 备份，再开启 **Watch Faces → Hermès**，执行 **Restart Carousel**。
7. 在手表表盘界面长按，滑到末尾点击 **＋**，添加 Hermès 表盘。
8. **完整关机、开机一次 Apple Watch**，解锁并重新连接手机，再检查编辑保存和手机 Watch App 显示。

本次最后解决编辑回退与手机不显示的是第 8 步。Restart Carousel 与整只手表完整重启是两种不同操作。

## 修改的核心

- 原 fork.2 配置 App Group 的循环跳过了 Watch App；本版为 Watch App ID 启用并关联同一个 App Group。
- 启用权限、关联组时传递正确的 watchOS 平台标记。
- 校验 Apple 下发的描述文件确实包含目标组，缺少时在签名和传输前停止。
- 对公开 anisette 服务返回的已知 macOS 10.12.6 客户端身份，使用原 fork.2 的兼容请求头组合；其他身份继续使用服务端信息。
- 提供 Windows CLI、IPA 检查和匿名登录服务诊断；保留上游的手机及手表安装流程。

## 后续使用

仅为使用爱马仕表盘时，额外开启 Hermès 即可，其他选项保持原始状态。保留初始备份；按 HermesEnabler 项目说明，删除 App 前先恢复备份。本命令行程序不提供后台自动续签，签名到期时间以实际描述文件为准。续签时沿用同一账户、团队与稳定的 Bundle ID。

程序在终端隐藏输入密码及验证码，不保存 Apple 账户密码；证书和 anisette 状态使用 Windows 凭据管理器。证书满额会停止，不会自动撤销旧证书。发布包不包含任何使用者的证书、描述文件或配对资料。

## 来源与许可

- 安装器基础：[AndreySemjonov/iloader](https://github.com/AndreySemjonov/iloader/tree/v2.3.4-fork.2)，基准提交 `2d13485ef2d7107b2bc2ec6cdb5f54ce5729ef70`。
- 原始 iloader / isideload：[nab138/iloader](https://github.com/nab138/iloader)、[nab138/isideload](https://github.com/nab138/isideload)。
- 手表安装流程沿用上游，包含 Data_C0re_（Rzbck）的相关贡献；见源码包内 `vendor/isideload/PROVENANCE.md`。
- 表盘工具：[HermesEnabler 1.4.1](https://github.com/pookjw/HermesEnabler/releases/tag/1.4.1)。
- 保留上游 MIT 许可和署名，见 [LICENSE](LICENSE)。

本项目提供可检查的权限申请、签名和安装流程；Apple 是否授予权限取决于实际账户与下发的描述文件。
