# 本项目的 Git 约定

> 一个人开发也要走完整流程。这不是形式主义——**你练的是以后和人协作时唯一正确的姿势。**

## 1. 一次性配置

```powershell
# 身份（已配置过可跳过）
git config --global user.name  "Ahilodz"
git config --global user.email "kk361103961@163.com"

# 换行符：交给 .gitattributes，关掉全局自动转换（否则两者互相打架）
git config --global core.autocrlf false

# 默认分支名
git config --global init.defaultBranch main

# 编辑器（不配置的话 git commit 会卡在奇怪的地方）
git config --global core.editor "code --wait"
```

> 为什么要 `core.autocrlf false`？
> 因为本仓库用 `.gitattributes` 精确声明了哪些文件用 CRLF、哪些用 LF、哪些是二进制。
> 全局自动转换会和它冲突，产生大量 "LF will be replaced by CRLF" 警告和意外的全文件 diff。

## 2. 分支命名

| 前缀 | 用途 | 例子 |
|---|---|---|
| `feat/` | 新功能 | `feat/unicode-injector` |
| `fix/` | 修 bug | `fix/focus-loss-pause` |
| `docs/` | 文档 | `docs/add-adr-006` |
| `refactor/` | 重构（不改行为） | `refactor/iinjector-interface` |
| `chore/` | 杂活（构建、依赖、配置） | `chore/setup-cmake` |

**规则**：`main` 分支永远可编译、可运行。所有开发在分支上进行，完成后合并回 `main`。

## 3. 提交信息格式

```
<类型>(<范围>): <一句话说明>

<可选正文：为什么这么改>
```

**类型**：`feat` / `fix` / `docs` / `refactor` / `test` / `chore` / `perf`

**示例**

```
feat(inject): 实现 UnicodeInjector 的 KEYEVENTF_UNICODE 发送
fix(session): 修复失焦暂停后修饰键未释放
docs(design): 补充 HID 下行协议的分帧说明
chore(build): 加入 CMakeLists 与 /W4 编译选项
```

**为什么不写"修了点东西"**：半年后你要靠这些信息判断"哪次改动引入了这个 bug"。
配合 `git bisect`，清晰的提交信息能把你从几小时的排查里救出来。

## 4. 什么该提交，什么不该

| ✅ 提交 | ❌ 不提交（已在 `.gitignore`） |
|---|---|
| 源代码 `.c .cpp .h .hpp` | 编译产物 `.exe .obj .o .axf .hex .bin` |
| 工程配置 `.ioc`（CubeMX）、`.uvprojx`、`CMakeLists.txt` | `.vs/`、`MDK-ARM/Objects/`、`build/` |
| 文档 `.md` | 证书私钥 `.pfx .pvk .key .pem` ⚠️ **泄露等于身份被盗用** |
| `.gitignore` / `.gitattributes` | 个人草稿（可用 `*.local.md` 排除） |

> **`*.ioc` 一定要提交**：它是 CubeMX 的工程描述，丢了要重新配置一小时。

## 5. 日常流程（背下来）

```powershell
git pull                              # 开工第一件事
git switch -c feat/xxx                # 开分支干活
# ... 写代码 ...
git status                            # 看看动了什么
git diff                              # 看看具体怎么动的
git add -A                            # 暂存
git commit -m "feat(xxx): 说明"        # 存档
git push -u origin feat/xxx           # 推到 GitHub
# 网页上开 Pull Request → Squash and merge → 删分支
git switch main
git pull
```

## 6. 提交粒度的建议

- **一次提交只做一件事**。改了三个不相关的东西，就分三次提交。
- **能编译通过再提交**。哪怕功能没做完，也要保证"这个存档点代码是能跑的"。
- **每天至少提交一次**。写一半下班前也要 `git stash` 或提交到 WIP 分支。

## 7. 后悔药

```powershell
git restore <file>              # 丢弃工作区里某个文件的改动
git restore --staged <file>     # 把文件撤出暂存区（改动还在）
git commit --amend --no-edit    # 修补上一次提交（还没 push 时用）
git reset --soft HEAD~1         # 撤销上一次提交，改动保留在暂存区
git reflog                      # 时光机：所有 HEAD 移动记录，误删分支也能找回
```

⚠️ **已经 push 到公共分支的提交，不要用 `reset --hard` 改写历史**，用 `git revert <hash>` 生成一个"反向提交"。

## 8. 版本标记

```powershell
git tag -a v0.1.0 -m "M1: PC 端最小窗口可用"
git push --tags
```

每完成一个里程碑打一个 tag，一年后回看就是一条清晰的时间线。
