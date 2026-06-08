# 个人 Fork 发版流程

这份文档记录如何让个人 fork 像官方仓库一样，通过 GitHub Releases 提供可下载安装包，并给每个构建赋予独立版本号。

## 当前发布机制

仓库已经包含官方同款 GitHub Actions 工作流：

- 工作流文件：`.github/workflows/release.yml`
- 触发方式：推送 `v*` 标签
- 输出位置：GitHub Releases
- 输出资产：macOS `.dmg`/`.zip`，Windows `.msi`/Portable `.zip`，Linux `.AppImage`/`.deb`/`.rpm`
- 额外资产：`latest.json`，用于 Tauri updater

也就是说，只要 fork 仓库开启 Actions，并满足签名密钥要求，推送版本标签后就会自动构建 Release。

## 版本号策略

建议个人 fork 使用能看出来源的版本号：

```text
v3.16.1-bt.1
v3.16.1-bt.2
```

含义：

- `3.16.1`：基于的上游 CC Switch 版本
- `bt`：个人 fork 标识
- `.1`、`.2`：个人构建序号

如果某个平台的打包或更新机制不接受带后缀的 SemVer 预发布版本，可以改用纯数字版本：

```text
v3.16.2
v3.16.3
```

这种情况下，Release 说明里要写清楚“基于上游 v3.16.1 + 本地改动”。

## 发版前更新版本号

发版前需要保持三个版本号一致：

- `package.json`
- `src-tauri/Cargo.toml`
- `src-tauri/tauri.conf.json`

例如发布 `3.16.1-bt.1` 时，三处都改成：

```json
"version": "3.16.1-bt.1"
```

Rust/TOML 中对应为：

```toml
version = "3.16.1-bt.1"
```

## GitHub Actions 前置条件

在 fork 仓库的 GitHub 页面确认：

1. Actions 已启用。
2. 仓库允许 `GITHUB_TOKEN` 写入 Releases。
3. 配置 Tauri updater 签名密钥 Secret：
   - `TAURI_SIGNING_PRIVATE_KEY`
   - `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`，如果私钥有密码

当前 fork 的 macOS 构建是无 Apple 签名、无公证打包，不需要 Apple 证书 Secret。下载后首次打开可能需要右键打开，或在系统设置的安全性页面中允许打开。

如果以后要启用 Apple 签名和公证，再恢复 workflow 中的签名步骤，并配置这些 Secret：

- `APPLE_CERTIFICATE`
- `APPLE_CERTIFICATE_PASSWORD`
- `APPLE_ID`
- `APPLE_PASSWORD`
- `APPLE_TEAM_ID`
- `KEYCHAIN_PASSWORD`

## 自动更新端点

当前 `src-tauri/tauri.conf.json` 的 updater endpoint 已指向个人 fork：

```json
"https://github.com/bt99bt/cc-switch/releases/latest/download/latest.json"
```

同时 `pubkey` 已更新为本 fork 的 Tauri updater 公钥。对应私钥存放在 GitHub Actions Secret：

```text
TAURI_SIGNING_PRIVATE_KEY
```

这个私钥不要提交到 git。丢失私钥后，旧版本应用无法校验后续自动更新包，只能重新下载安装。

## 发版命令

确认工作区干净后：

```bash
git status --short
```

提交改动：

```bash
git add .
git commit -m "chore: prepare local fork release"
```

创建并推送版本标签：

```bash
git tag v3.16.1-bt.1
git push origin codex/local-custom
git push origin v3.16.1-bt.1
```

推送标签后，GitHub Actions 会自动运行 Release workflow。

## 下载位置

发布成功后，在这里下载：

```text
https://github.com/bt99bt/cc-switch/releases
```

Release 资产命名会类似：

```text
CC-Switch-v3.16.1-bt.1-Windows.msi
CC-Switch-v3.16.1-bt.1-Windows-Portable.zip
CC-Switch-v3.16.1-bt.1-macOS.dmg
CC-Switch-v3.16.1-bt.1-Linux-x86_64.AppImage
```

## 跟上游同步后的发版

每次从官方仓库同步后，如果要发布新的个人构建：

1. 先按 `docs/local-development-workflow.md` 同步 `main` 并 rebase 自定义分支。
2. 确认本地功能和测试状态。
3. 更新版本号。
4. 提交、打 tag、推送 tag。

这样 release 历史就能清楚表达“基于哪个上游版本 + 第几次个人构建”。
