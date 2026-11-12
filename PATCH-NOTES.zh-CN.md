# 我们修改 iloader 的思路

基准是 `AndreySemjonov/iloader v2.3.4-fork.2`，提交 `2d13485ef2d7107b2bc2ec6cdb5f54ce5729ef70`。本版名称为 `2.3.4-watchgroup.2`。

## 一、从“能安装但不能用”追到签名权限

手机侧载完成、手表 App 安装完成、手表 App 获得所需权限，是不同层面的结果。本次手表 App 已可启动，却提示 MobileGestalt 访问失败，因此进一步检查 App Group 配置流程。

原 fork.2 的 `vendor/isideload/isideload/src/sideload/sideloader.rs` 在遍历 App ID 配置组权限时，有这样的逻辑：

```rust
if watch_bundle_ids.contains(&app_id.identifier) {
    continue;
}
```

因此 Watch bundle 没有进入后续组权限启用与关联流程。本版识别 Watch ID 后，设置 `DeveloperDeviceType::Watchos`，然后继续执行配置。

## 二、补齐启用、关联、平台参数

修改了三个环节：

1. `dev/app_ids.rs`：增加 `ensure_group_feature_for_platform`，为 Watch App ID 启用 App Groups 功能，同时传入平台信息。未返回该功能字段时按尚未启用处理。
2. `dev/app_groups.rs`：关联 App Group 的请求体调用 `apply_platform_to_body`，带上与 Watch App ID 匹配的平台标记。
3. `sideload/sideloader.rs`：为手机和手表 App 关联同一个组。手机特有的增加内存限制操作继续只用于手机 App。

这里的核心是让 Apple 侧 App ID 配置和稍后签发的描述文件一致。

## 三、检查 Apple 真正下发了什么

下载描述文件后，读取其中的：

```text
com.apple.security.application-groups
```

新增 `validate_watch_app_group`，要求它是数组，并且包含此次请求的准确组标识。字段缺失、类型错误或组不匹配，都会在签名和传输之前停止。

通过时打印：

```text
Watch App Group verified in Apple-issued provisioning profile
```

这项校验把“申请了某项权限”和“实际拿到权限”区分开。Apple 没有签发的权限，不能靠本地增加一个 entitlement 字段得到。

## 四、第二版修复登录兼容性遗漏

第一版从 anisette 服务的 `/v3/client_info` 动态获取客户端信息。实际服务返回的是一组已知旧身份：macOS 10.12.6。随后登录返回 `-23782`，要求 macOS 10.13 或更高版本。

第二版在 `anisette/remote_v3/mod.rs` 中增加针对性兼容处理：

- 精确匹配这组已知旧身份时，恢复原 fork.2 的 `client_info` 与 `User-Agent` 组合。
- 其他有效身份继续使用服务端返回值。
- 同一次登录缓存这一组合，切换服务地址时清空缓存。
- 空值、无效 JSON、HTTP 错误继续报错。

另加 `--check-login-service`，只获取公开服务信息，便于在输入账户前诊断网络和身份兼容性。TLS 校验和 Apple 账户认证流程仍然执行。

## 五、整理成 Windows CLI

增加独立的 `iloader-cli` Cargo 包，调用 vendored isideload 后端，保留上游手机和手表安装链路。提供 IPA 结构与 SHA256 检查、USB 设备选择、隐藏输入密码和验证码、同账户证书复用、明确的分阶段错误。

证书名额不足时使用 `MaxCertsBehavior::Error` 停止。当前 CLI 没有后台自动续签功能。

## 六、验证与实机结果

构建时 CLI 5 项、后端 18 项测试通过，共 23 项；另检查了原始 IPA、公开登录服务兼容处理和启动脚本。

`BUILD.original.json` 保存当时的构建记录，其中真实账户登录和手表安装标记仍为 false，因为打包阶段尚未进行。随后由使用者完成实机操作，并在 2026-11-12 确认能添加表盘；完整重启手表后，编辑保存与手机端显示均正常。两类证据按发生顺序分别保留。

安装器补丁处理签名权限和登录；表盘编辑与手机显示的最后恢复来自完整重启手表。可在 [watch-app-group.patch](watch-app-group.patch) 中查看四个后端文件的差异；新增 CLI 的完整源码位于源码包。
