# 从源码编译

下载本仓库 Releases 中的 `iloader-2.3.4-watchgroup.2-source.zip` 并完整解压。源码包含 CLI 与其 `vendor/isideload` 路径依赖，无需额外复制另一个 isideload 仓库。

## 本次构建环境

| 项目 | 记录 |
| --- | --- |
| Rust | 1.99.1 |
| 目标 | x86_64-pc-windows-gnu |
| GCC | 13.2.0，来自 Strawberry Perl portable 5.42.3.1 |
| Release 配置 | strip = true，lto = thin |
| apple-codesign | Cargo.lock 固定至 5da0b8df9b202434b1eff2700059b1ee142592ac |

安装 Git、匹配的 Rust 工具链与 MinGW-w64 GCC，将相应工具目录加入当前终端 PATH。GCC 是构建依赖，使用预编译 EXE 无需安装 Rust 或 GCC。

在源码根目录运行：

```powershell
cargo test --manifest-path iloader-cli/Cargo.toml --locked
cargo test --manifest-path iloader-cli/Cargo.toml -p isideload --lib --locked
cargo build --release --manifest-path iloader-cli/Cargo.toml --locked
```

上述本次构建使用 GNU 默认目标。若机器默认不是 GNU，应选择对应工具链或为命令显式指定 `--target x86_64-pc-windows-gnu`。

首次构建需要下载 Cargo 依赖；只有依赖已经完整缓存时才使用 `--offline`。保留 Cargo.lock，避免排错时引入无关依赖更新。

未指定 `--target` 且没有自定义 CARGO_TARGET_DIR 时，EXE 通常在 `iloader-cli/target/release/`；显式指定目标时通常在 `iloader-cli/target/x86_64-pc-windows-gnu/release/`。

## 原二进制与本次发布包

本次发布沿用已经实机使用的 EXE，没有为制作文档重新编译。EXE 的 SHA256 为：

```text
f10180a7e53c265e3f5de7cb2e8e7e05c308ecee1316775bbc20416fea1cf44b
```

源码中 CLI 的两个 Rust 文件、两个 Cargo 文件，以及四个后端修改文件与原构建记录中的哈希逐项核对。说明文档和打包结构为公开发布整理过，因此公开 ZIP 的哈希与原本地 ZIP 不同。不能把自行编译所得 EXE 的字节哈希必然一致作为保证。

原始记录见 `BUILD.original.json`。发布时另生成 `SHA256SUMS.txt` 用于下载文件完整性核对。
