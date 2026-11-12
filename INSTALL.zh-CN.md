# Windows 详细安装教程

对应程序：`2.3.4-watchgroup.2`。以下是本次成功顺序，建议逐项完成。

## 1. 准备电脑、手机和手表

电脑使用 Windows x64。本次通过 iTunes 安装的 Apple 设备服务识别 iPhone。先用可以传输数据的数据线连接手机，解锁，在手机上选择信任电脑并按提示输入锁屏密码。

打开 iTunes，确认能进入 iPhone 设备详情页，看到型号、容量等信息。手机可以充电，甚至弹出过信任提示，都不等于 Windows 设备服务已经正常识别。设备详情页确认成功后再继续。

Apple Watch 应已与这台 iPhone 配对。手表戴在手腕上并解锁，靠近手机；安装期间两台设备都保持解锁。若设备要求开发者模式，在相应设备设置中按提示开启、重启并确认。开发者模式是设备开关；付费开发者会员是账户服务，两者不能混为一谈。

## 2. 下载正确文件

从本仓库 Releases 下载：

`iloader-2.3.4-watchgroup.2-windows-x64.zip`

完整解压到一个普通文件夹。文件夹中应有 `iloader-cli.exe`、`Start.cmd`、`Check.cmd`、`Check-login.cmd` 和说明文件。不要直接在压缩包预览里运行。

从 [HermesEnabler 1.4.1 官方发布页](https://github.com/pookjw/HermesEnabler/releases/tag/1.4.1) 的 Assets 下载 **HermesEnabler.ipa**。两个 Source code 压缩包是源码，不能代替 IPA 安装。

把 `HermesEnabler.ipa` 放到 `iloader-cli.exe` 同一目录，保持文件名。该版本官方 IPA 的 SHA256 为：

```text
5b7894cf38ad7f4cbe08743e4fed41db8b3f06b133c90db8aa02f43ef1e66092
```

双击 `Check.cmd`，本次应显示 `IPA checked: 1 embedded Watch app(s)`、SHA256 和 `CHECK OK`。这一步只检查文件，不登录 Apple 账户、不访问设备。程序打印哈希供人工核对，并非锁定只允许某个版本的 IPA。

## 3. 运行新版程序

退出旧版 iloader，包括托盘后台进程，并关闭正在操作设备的其他侧载工具。双击 `Start.cmd`，第一行确认版本为 `2.3.4-watchgroup.2`。

如果没有把 IPA 放在同一目录，程序会要求输入路径；可将 IPA 拖入这个程序窗口后回车。用命令行也可以显式传入路径：

```powershell
.\iloader-cli.exe "C:\Apps\HermesEnabler.ipa"
```

随后应显示 USB 设备可用。若连接了多个手机，按程序提示选择。

## 4. 登录自己的 Apple 账户

使用之前成功安装时同一个 Apple 账户，优先填写该账户实际的登录邮箱。不是随意找一个邮箱；号码账户的格式也不宜反复猜测。

密码输入时不显示字符，属于正常行为。输入后回车，再按提示输入受信任设备上的验证码。程序也提供 `device`、`sms` 和 `cancel` 选项。密码和验证码无需发给其他人。

如果账户有多个开发者团队，选择上次安装使用的团队。程序会尽量复用原 fork.2 存放的签名证书。遇到证书满额会停止，不会自动撤销正在使用的证书。

默认 anisette 服务是 `https://mac-ani.jkcoxson.com`，这是登录所用的第三方辅助服务。可双击 `Check-login.cmd` 单独检查公开客户端信息；通过此诊断只代表该接口返回有效信息，不等于 Apple 账户登录成功。

## 5. 核对权限，再等待安装

注意日志中的这行：

```text
Watch App Group verified in Apple-issued provisioning profile
```

它表示程序已在 Apple 下发的手表描述文件里找到了准确的 App Group 授权。之后程序继续签名、安装手机 App，并通过手机代理完成 Watch App 安装。

最终应看到：

```text
DONE: iPhone and Apple Watch installation completed.
```

`DONE` 是安装流程完成的标志。下一步还需要在手表上打开 App，确认功能可用。若出现 `device is locked`，检查手机和手表两端的解锁状态。

## 6. 在手表先备份，再启用 Hermès

在手表应用列表打开 **HermesEnabler2**。首次使用先创建初始 MobileGestalt 备份，保留这份原始备份。

进入 **Watch Faces / 表盘**，新增开启 **Hermès**，其他功能先保留原始状态。回到应用首页，执行 **Restart Carousel / 重启 Carousel**。

如果仍出现 MobileGestalt 访问失败，不要继续反复切换功能；回到排查文档，核对权限验证行、手表上实际安装的版本和系统兼容性。

## 7. 在手表添加表盘

回到显示时间的表盘界面：

1. 长按当前表盘。
2. 向左滑动到末尾。
3. 点击 **＋ / 新增**。
4. 找到 Hermès 表盘，选择并添加。

本次从手表本身完成添加，之后再检查手机 Watch App。

## 8. 完整重启 Apple Watch

本次添加表盘后曾出现两种问题：进入编辑后颜色、样式或功能无法保存；手机 Watch App 不显示爱马仕表盘。

将 Apple Watch **正常关机，再按住侧边按钮开机**。重新解锁，等待与 iPhone 恢复连接，再打开手机 Watch App。本次完整重启后，两项问题都恢复正常。

Restart Carousel 只重新加载手表界面；本次成功记录中的最后一步是整只手表关机、开机。

## 9. 最后检查

- 手表能打开 HermesEnabler2，能创建初始备份。
- 手表已经添加 Hermès 表盘。
- 修改颜色、样式或功能后退出，再进入仍保留修改。
- 手机 Watch App 能显示已添加的爱马仕表盘。

本次这四项均由使用者确认。其他设备出现不同结果时，按实际错误排查。

## 10. 续签、回退和卸载

本命令行版没有后台自动续签；到期时间看实际签发的描述文件。重新签名时尽量使用同一账户、团队和稳定的 Bundle ID，先保留现有 App 与备份。

按 HermesEnabler 项目说明，删除 App 之前先在 App 内恢复备份。系统升级前重新核对原项目兼容性。
