# 本地自定义开发流程

这份文档记录当前 fork 的维护方式，目标是：本地可以持续做小功能改动，同时尽量轻松地跟上官方仓库进度。

## 仓库关系

- 官方仓库：<https://github.com/farion1231/cc-switch>
- 我的 fork：<https://github.com/bt99bt/cc-switch>
- 本地 `origin`：指向我的 fork，用来推送自己的分支和改动。
- 本地 `upstream`：指向官方仓库，用来拉取官方更新。

推荐原则：

- `main` 分支保持干净，只用于同步官方 `upstream/main`。
- 自定义功能放在独立分支，例如 `codex/local-custom` 或 `codex/<feature-name>`。
- 不直接在 `main` 上做功能改动。

## 同步官方 main

当官方仓库有新进度时，先更新本地 `main`：

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

如果希望历史更线性，也可以把 `merge` 换成 `rebase`：

```bash
git fetch upstream
git checkout main
git rebase upstream/main
git push origin main
```

## 开发一个小功能

从最新的 `main` 切出功能分支：

```bash
git checkout main
git pull --ff-only origin main
git checkout -b codex/my-small-feature
```

修改代码后提交：

```bash
git add .
git commit -m "feat: add my small feature"
git push -u origin codex/my-small-feature
```

## 让自定义分支跟上官方进度

先同步 `main`：

```bash
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

再把自定义分支接到最新 `main` 上：

```bash
git checkout codex/local-custom
git rebase main
git push --force-with-lease
```

说明：

- `rebase` 会把自定义提交重新放到最新 `main` 后面，历史更清楚。
- `--force-with-lease` 用于更新已经 rebase 过的远程分支，比普通 `--force` 更安全。
- 如果发生冲突，先解决冲突，再执行：

```bash
git add <resolved-files>
git rebase --continue
```

## 长期维护建议

- 常驻私人改动可以放在 `codex/local-custom`。
- 准备提交给官方的功能，单独开 `codex/<feature-name>`，方便后续提 PR。
- 每次开始新功能前，先让 `main` 对齐 `upstream/main`，再从 `main` 开分支。
- 遇到官方大版本更新时，先只同步并跑测试，确认基础状态正常后再 rebase 自定义分支。
