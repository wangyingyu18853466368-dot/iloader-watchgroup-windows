# 踩坑记录：按错误发生的阶段排查

这份记录保留实际经历与修复边界。看到报错时，先区分设备连接、账户登录、签名权限、手表安装和表盘刷新，避免在错误的环节反复尝试。

## 1. 手机能充电，也点过信任，但电脑没有设备

**当时现象：**Sideloadly 显示 `no devices detected`。

**处理：**先解决 Windows 的 Apple 设备服务和 USB 识别。本次在 iTunes 中能看到 iPhone 详情页后，侧载工具才识别到设备。充电不能代替数据连接验证。只有设备详情可见后再启动本项目。

## 2. Sideloadly 报 App ID 不可用

**报错：**`9401 ... An App ID with Identifier 'com.example.HermesEnabler' is not available`。

**当时处理：**改为唯一的应用标识后完成了手机安装，但手表仍没有 App。它解决的是 App ID 冲突；手机安装完成不能证明手表安装和权限也完整。

**本项目建议：**使用原版 IPA，让安装流程协调手机与 Watch 的标识关系。只改其中一个标识可能破坏伴生关系；本项目在安装前会检查 IPA 中的手机、Watch 标识是否匹配。

## 3. 手机有 App，Watch 的“可用 App”和“已安装”都没有

**当时现象：**手机侧载显示完成，手表列表找不到 HermesEnabler2。

**转折：**改用带 Apple Watch 伴生安装支持的 iloader fork.2，并核对原版 IPA 内含 Watch App。该安装链路负责经 iPhone 代理安装手表 App。不能把本次经历外推为所有版本、账户下 Sideloadly 都无法安装手表 App。

## 4. iloader 登录失败：-80045 或 provisioning 超时

`-80045` 本身不足以确定是密码、服务还是账户状态问题，要展开具体错误。

本次详细堆栈曾包含：

```text
Failed to get anisette data for login
Failed to provision
Failed to connect to provisioning socket
deadline has elapsed
```

这次失败点是登录辅助服务连接超时。检查网络与该服务是否可达；本次随后使用可连接的 `mac-ani.jkcoxson.com` 继续。服务可用性会变化，不保证持续可用。

`Check-login.cmd` 只检查公开客户端信息接口。它通过后，实际 provisioning 或 Apple 登录仍可能失败。不要仅凭该结果认定账户权限已正常。

## 5. Apple Developer 网页提示“账户暂时不符合条件”

本次在网页登录环节见过此提示，仅凭这句话无法确定原因。该网页状态与手表 App 是否具有 App Group 授权不能直接画等号；应查看安装器申请权限、下载描述文件的实际结果。

本次没有完整核验账户会员分类，因此不据此承诺所有免费账户都适用，也不把租借开发者账号作为必要步骤。

## 6. 手表安装失败：device is locked

堆栈中的 companion proxy 配对失败并指出设备锁定时，检查 **iPhone 和 Apple Watch 两端**。手表戴在手腕上、输入密码解锁，靠近手机后重试。手表离腕后再次上锁也会打断流程。

## 7. 手表要求开发者模式

按手表系统提示开启、重启并确认；手机端若也要求，则同样完成。不能用开启手机开发者模式代替手表侧的确认。本次整个过程在 Windows 完成。

## 8. 装上了，但提示 MobileGestalt 访问失败

手表提示 App Group 可能缺失或系统不兼容。检查 fork.2 源码发现，配置 App Group 时跳过了 Watch App。

本版补齐 Watch App ID 的 App Groups 开关、组关联和 watchOS 平台参数，再检查 Apple 下发的描述文件。详见 [修复原理](PATCH-NOTES.zh-CN.md)。

若仍失败，先记录是否出现 `Watch App Group verified in Apple-issued provisioning profile`：

- 没出现且程序提示无法授权：保留 Apple 返回的具体原因；本地补写 entitlement 不能替代 Apple 授权。
- 已出现，但手表仍报错：继续核对手表上实际启动的 App、系统版本、配置和签名结果。这个提示同时列出多种可能原因，不能直接判断为唯一原因。

## 9. 第一版本地 CLI 报 -23782，要求 Mac / macOS 10.13+

第一版直接采用 anisette 服务返回的 macOS 10.12.6 客户端信息，随后 Apple 拒绝登录。

第二版 `2.3.4-watchgroup.2` 对已知旧身份恢复原 fork.2 的兼容请求头组合。启动应显示正确版本；遇到这一旧身份时，会打印“已启用 fork.2 登录兼容信息”。该调整发生在登录阶段，与 Watch App Group 是两处独立修复。

如果新版仍出现相同错误，记录新堆栈，核对版本和服务响应，不必为登录问题删除手表 App。

## 10. 表盘能添加，但编辑不保存；手机 Watch App 不显示

本次真实发生过，使用者反馈为：颜色、样式或功能无法保存；手机端不显示包括已经添加的爱马仕表盘。

**最终有效操作：完整关机、开机一次 Apple Watch。** 解锁并与手机重新连接后，两项均恢复正常。这一步不需要重新编译安装器。

社区相关反馈：[编辑回退 #36](https://github.com/pookjw/HermesEnabler/issues/36)、[完整重启后的反馈 #37](https://github.com/pookjw/HermesEnabler/issues/37#issuecomment-6791221671)、[手机显示 #49](https://github.com/pookjw/HermesEnabler/issues/49)。这些是其他使用者报告，可辅助排查，不构成所有系统必然有效的保证。

## 11. 哪些选项先不折腾

只需要 Hermès 时，新增开启 Hermès，其他设置保持原始状态。先保留初始 MobileGestalt 备份。不要把原本开启的硬件功能全部关掉。

本次不建议额外开启 Ultra Faces 或改分辨率：项目与社区有特定表盘兼容性、界面缩放和裁切异常反馈。需要额外功能时逐项验证，不把“全部打开”当作标准安装步骤。

## 提交问题时提供什么

提供程序版本、iOS/watchOS 版本和构建号、错误全文、卡在哪一步、是否出现 App Group 验证成功行，以及完整重启后的结果。分享日志或截图前遮住账户、手机号、设备标识；不要上传密码、验证码、证书私钥或配对文件。
