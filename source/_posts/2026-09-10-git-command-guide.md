---
title: Git 指令实战大全：从入门到团队协作，一篇打通所有常用操作
date: 2026-09-10 12:00:00
categories:
  - 技术
  - 工具
tags:
  - Git
  - 版本控制
  - 开发工具
---

## 前言

Git 是现代软件开发的基石，无论是个人项目还是团队协作，掌握 Git 都是程序员的必备技能。本文从核心概念出发，系统梳理日常开发、分支管理、远程协作、版本回滚、冲突解决等全场景指令，每一条命令都附带场景、示例、参数详解与避坑提示，兼顾新手入门与进阶查用，建议收藏常备。

<!-- more -->

---

## 一、Git 基础认知与环境初始化

### 1.1 四大工作区（必须先理解的核心模型）

Git 的所有操作本质上都是在四个区域之间搬运代码，理解这个模型是后续所有命令的基础。

| 区域 | 英文 | 含义 | 对应命令入口 |
| --- | --- | --- | --- |
| 工作区 | Working Directory | 你在编辑器里看到、正在修改的文件 | 直接编辑文件 |
| 暂存区 | Staging Area / Index | 下次提交的"候选清单"，git add 后进入 | git add |
| 本地仓库 | Local Repository | 本机的版本历史，git commit 后进入 | git commit |
| 远程仓库 | Remote Repository | GitHub/GitLab/Gitee 上的共享仓库 | git push / git pull |

**数据流向主线：**

```text
# 提交流向
工作区 --git add--> 暂存区 --git commit--> 本地仓库 --git push--> 远程仓库

# 回退流向
工作区 <--git restore-- 暂存区 <--git reset-- 本地仓库 <--git pull/fetch-- 远程仓库
```

### 1.2 全局环境配置

**使用场景**：安装 Git 后第一步，设置身份信息，所有提交记录都会携带该信息。

**核心命令与示例：**

```bash
# 设置全局用户名
git config --global user.name "你的用户名"

# 设置全局邮箱
git config --global user.email "你的邮箱@example.com"

# 查看所有全局配置
git config --global --list

# 查看单项配置
git config user.name

# 删除某项配置
git config --global --unset user.name
```

**参数解释：**

- `--global`：全局配置，对当前系统用户所有仓库生效；去掉则只对当前仓库生效。

**避坑注意点：**

- 公司项目建议使用公司邮箱配置，个人项目使用个人邮箱，避免提交记录身份混乱。
- 配置错误不会报错，只会体现在提交记录里，建议配置后立刻 `git config --list` 核对。

### 1.3 本地仓库初始化

**使用场景**：本地新建项目，从零开始用 Git 管理。

**核心命令：**

```bash
# 在当前目录初始化 Git 仓库
git init
```

**执行后效果**：当前目录生成隐藏文件夹 `.git`，包含版本管理所有元数据。

**避坑注意点：**

- 不要在仓库内部再 `git init`，会造成嵌套仓库，导致版本管理混乱。
- 删除 `.git` 文件夹即可移除 Git 管理，文件本身不会丢失。

### 1.4 克隆远程仓库

**使用场景**：拿到已有项目地址，下载到本地开发。

**核心命令与示例：**

```bash
# 普通克隆（默认主分支，完整历史）
git clone https://github.com/xxx/xxx.git

# 指定分支克隆
git clone -b dev https://github.com/xxx/xxx.git

# 浅克隆（只拉取最近一次提交，速度快，体积小）
git clone --depth=1 https://github.com/xxx/xxx.git
```

**参数解释：**

- `-b 分支名`：克隆指定分支，而非默认主分支。
- `--depth=1`：只拉取最新 1 次提交历史，适合只需要代码、不需要完整历史的场景。

**避坑注意点：**

- 浅克隆无法查看历史提交、无法切换旧版本，需要完整历史时不要加 `--depth`。
- 克隆后自动关联远程仓库，别名默认为 `origin`。

---

## 二、日常开发基础核心指令

### 2.1 git status 查看状态

**使用场景**：修改文件后、提交前，随时查看当前工作区和暂存区状态。

**核心命令：**

```bash
# 完整状态
git status

# 精简状态（一行显示）
git status -s
```

**状态标识说明：**

- 红色：文件在工作区，未加入暂存区
- 绿色：文件已加入暂存区，待提交
- `M`：修改、`A`：新增、`D`：删除、`??`：未跟踪

### 2.2 git add 添加到暂存区

**使用场景**：文件修改完成，准备提交前，先加入暂存区。

**核心命令与示例：**

```bash
# 添加单个文件
git add index.html

# 添加多个文件（空格分隔）
git add index.html style.css

# 添加整个目录
git add src/

# 添加所有已跟踪文件的修改（不包含新增未跟踪文件）
git add -u

# 添加所有文件（包含新增、修改、删除）
git add .

# 交互式添加，逐个选择文件
git add -i
```

**参数解释：**

- `.`：当前目录及所有子目录。
- `-u`：update，只处理已经被 Git 跟踪的文件，忽略新增未跟踪文件。

**避坑注意点：**

- `git add .` 会把所有文件加入暂存区，包括临时文件、日志文件，建议配置 `.gitignore` 后再使用。
- 添加后发现改错了，用 `git restore --staged 文件名` 撤出暂存区。

### 2.3 git commit 提交到本地仓库

**使用场景**：暂存区内容确认无误，生成一次版本提交。

**核心命令与示例：**

```bash
# 常规提交（会弹出编辑器写注释）
git commit

# 直接附带提交信息
git commit -m "feat: 新增用户登录接口"

# 暂存+提交 一步完成（只包含已跟踪文件）
git commit -am "fix: 修复登录页验证码不显示"

# 重写最近一次提交（修改注释或补充文件）
git commit --amend -m "feat: 新增用户登录接口（补充校验）"
```

**参数详解：**

- `-m`：直接附带提交信息。
- `-a`：自动暂存所有已跟踪文件的修改与删除（不含新文件）。
- `--amend`：重写最近一次提交，若该提交已 push，慎用。

### 2.4 Conventional Commits 提交规范

团队协作必须统一提交注释格式，主流规范如下：

```text
<type>(<scope>): <subject>

<body>

<footer>
```

| type | 含义 |
| --- | --- |
| feat | 新功能 |
| fix | 修复 Bug |
| docs | 文档变更 |
| style | 代码格式（不影响逻辑） |
| refactor | 重构（非新功能、非修复） |
| perf | 性能优化 |
| test | 测试相关 |
| chore | 构建/工具/依赖等辅助变更 |

**避坑注意点：**

- `--amend` 只能修改本地未推送的提交，已推送到远程的提交不要修改，会造成历史不一致。

### 2.5 git log 查看提交日志

**使用场景**：查看提交历史、定位版本、查找某次提交。

**核心命令与示例：**

```bash
# 完整日志（默认分页）
git log

# 单行精简日志
git log --oneline

# 显示最近 n 条
git log -5

# 显示提交差异详情
git log -p

# 按作者过滤
git log --author="张三"

# 查看指定文件的提交历史
git log --oneline index.html
```

**参数解释：**

- `--oneline`：每条提交一行显示，只显示哈希前7位和注释，最常用。
- `-p`：patch，显示每次提交的具体代码改动。

### 2.6 git diff 对比代码差异

**使用场景**：提交前查看具体改了什么，对比不同区域、不同版本的差异。

**核心命令与示例：**

```bash
# 对比工作区 vs 暂存区
git diff

# 对比暂存区 vs 本地仓库
git diff --cached

# 对比工作区 vs 本地仓库最新提交
git diff HEAD

# 对比两个提交
git diff abc123 def456

# 对比两个分支
git diff main dev
```

**避坑注意点：**

- `git diff` 默认只看工作区和暂存区差异，已经 add 的文件看不到，需要加 `--cached`。

---

## 三、分支管理全套指令（开发核心）

### 3.1 分支原理

分支本质上是指向某个提交的可变指针，创建分支就是新建一个指针，互不影响，并行开发。

### 3.2 分支基础操作

**使用场景**：开发新功能、修复 Bug，基于主分支创建独立分支。

**核心命令与示例：**

```bash
# 查看本地分支
git branch

# 查看远程分支
git branch -r

# 查看所有分支（本地+远程）
git branch -a

# 创建分支
git branch feature/login

# 切换分支
git checkout feature/login

# 创建并切换分支（常用）
git checkout -b feature/login

# 另一种创建切换方式（Git 2.23+）
git switch -c feature/login

# 重命名分支
git branch -m oldname newname

# 删除本地分支（已合并）
git branch -d feature/login

# 强制删除本地分支（未合并也删）
git branch -D feature/login

# 删除远程分支
git push origin -d feature/login
```

**参数解释：**

- `-d`：delete，只能删除已合并的分支，安全。
- `-D`：强制删除，未合并也会删除，慎用。

### 3.3 git merge 分支合并

**使用场景**：功能开发完成，合并回主分支。

**核心命令与示例：**

```bash
# 切换到目标分支（通常是主分支）
git checkout main

# 合并指定分支到当前分支
git merge feature/login

# 快速合并（如果可以快进）
git merge --ff-only feature/login

# 禁止快进，生成合并提交
git merge --no-ff feature/login
```

**参数解释：**

- `--ff-only`：只有能快进时才合并，否则失败。
- `--no-ff`：强制生成一次合并提交，保留分支历史轨迹，团队协作推荐。

**合并冲突**：同一文件同一位置被两个分支修改，会产生冲突，需要手动解决后再提交。

### 3.4 git rebase 变基

**使用场景**：让提交历史变成一条直线，保持整洁，通常用于个人开发分支同步主分支更新。

**核心命令与示例：**

```bash
# 将当前分支变基到目标分支
git rebase main

# 交互式变基（合并、修改、删除提交）
git rebase -i HEAD~3
```

**交互式变基常用指令：**

- `pick`：保留该提交
- `squash`：合并到上一个提交
- `reword`：修改提交注释
- `drop`：删除该提交

### 3.5 merge 与 rebase 深度对比

| 维度 | merge | rebase |
| --- | --- | --- |
| 历史形态 | 保留分支轨迹，有合并节点，历史完整 | 线性历史，无额外节点，整洁干净 |
| 提交哈希 | 不变 | 变基后的提交哈希全部改变 |
| 冲突处理 | 一次解决 | 逐个提交解决，可能多次冲突 |
| 适用场景 | 公共分支合并、多人协作 | 个人开发分支同步主分支 |
| 风险 | 低，历史完整可追溯 | 高，修改历史，公共分支禁用 |

**一句话总结：公共分支用 merge 保真实，个人分支用 rebase 保整洁。**

**避坑注意点：**

- **绝对不要在公共分支（main/dev）上执行 rebase**，会修改所有人的提交历史，造成团队灾难。
- 已经 push 到远程的提交，不要本地 rebase 后强推，除非确认只有你一个人用这个分支。

---

## 四、远程仓库协作指令

### 4.1 远程仓库关联与管理

**使用场景**：本地仓库推送到 GitHub/Gitee、更换远程地址、一个项目推送多个远程。

**核心命令与示例：**

```bash
# 查看远程仓库（简略）
git remote

# 查看远程仓库及地址（详细）
git remote -v

# 关联远程仓库
git remote add origin https://github.com/xxx/xxx.git

# 修改远程地址
git remote set-url origin https://github.com/xxx/new.git

# 移除远程仓库
git remote remove origin
```

### 4.2 git fetch 与 git pull

**使用场景**：同步远程仓库的更新到本地。

**核心命令与示例：**

```bash
# 拉取远程所有更新（只下载，不合并）
git fetch origin

# 拉取指定分支
git fetch origin dev

# 拉取并合并到当前分支
git pull origin dev

# 拉取并以 rebase 方式合并
git pull --rebase origin dev
```

### 4.3 pull 与 fetch 核心区别

| 命令 | 动作 | 对本地代码影响 | 安全性 |
| --- | --- | --- | --- |
| git fetch | 只下载远程更新到本地仓库，不合并 | 不修改工作区代码 | 高 |
| git pull | fetch + merge，下载并自动合并 | 直接修改工作区，可能产生冲突 | 中 |

**最佳实践：** 先 `git fetch` 查看更新，确认没问题再 `git merge`，比直接 `git pull` 更可控。

### 4.4 git push 推送代码

**使用场景**：本地提交完成，推送到远程仓库共享。

**核心命令与示例：**

```bash
# 推送到远程指定分支
git push origin main

# 设置上游分支（之后可直接 git push）
git push -u origin main

# 强制推送（慎用）
git push -f origin feature/login
```

**参数解释：**

- `-u`：`--set-upstream`，绑定本地分支与远程分支，后续直接 `git push` 即可。
- `-f`：`--force`，强制覆盖远程，风险极高。

**避坑注意点：**

- 公共主分支绝对禁止强制推送。
- 个人分支强推后，如果有其他人基于该分支开发，会造成对方历史混乱。

---

## 五、git stash 代码暂存全套用法

### 5.1 适用场景

写到一半的代码不想提交，但需要临时切换分支、紧急修 Bug，用 stash 临时储藏起来。

### 5.2 核心命令

```bash
# 暂存当前修改
git stash

# 带备注暂存（推荐，方便识别）
git stash push -m "写到一半的登录表单"

# 暂存指定文件
git stash push src/login.js

# 查看暂存列表
git stash list

# 恢复最近一次暂存（恢复后删除该暂存）
git stash pop

# 恢复指定暂存
git stash pop stash@{1}

# 恢复但不删除暂存
git stash apply

# 删除指定暂存
git stash drop stash@{1}

# 清空所有暂存
git stash clear
```

**避坑注意点：**

- `git stash pop` 恢复后会删除暂存记录，`apply` 保留，根据场景选择。
- 暂存是本地的，不会推送到远程，换电脑看不到。
- 不要长期堆积 stash，及时清理，避免混乱。

---

## 六、代码撤销、回滚与还原（重点避坑）

### 6.1 工作区撤销（未 add）

**使用场景**：文件改乱了，想恢复到修改前的样子。

```bash
# 撤销单个文件修改
git restore index.html

# 撤销所有文件修改
git restore .
```

> 
> 注意：执行后工作区修改**彻底丢失**，无法恢复，谨慎使用。

### 6.2 暂存区撤销（已 add 未 commit）

**使用场景**：add 错了文件，想撤出暂存区。

```bash
# 撤出单个文件
git restore --staged index.html

# 撤出所有
git restore --staged .
```

> 
> 只撤出暂存区，工作区修改保留，不会丢代码。

### 6.3 git reset 回滚提交（已 commit）

**使用场景**：提交错了，想回退到某个版本。

#### 三种模式对比

| 模式 | 工作区 | 暂存区 | 提交历史 | 适用场景 |
| --- | --- | --- | --- | --- |
| --soft | 不变 | 不变 | 回退 | 重新整理提交 |
| --mixed（默认） | 不变 | 回退 | 回退 | 撤销提交，重新修改提交 |
| --hard | 回退 | 回退 | 回退 | 彻底丢弃所有修改，完全回到某个版本 |

**核心命令示例：**

```bash
# 回退一次提交（默认 mixed）
git reset HEAD~1

# 软回退（提交撤销，代码保留在暂存区）
git reset --soft HEAD~1

# 硬回退（代码全部丢弃，彻底回到上一版本）
git reset --hard HEAD~1

# 回退到指定提交
git reset --hard abc123
```

**避坑注意点：**

- `--hard` 会删除工作区所有未提交修改，**不可逆**，执行前确认代码不需要了。
- 已经 push 到远程的提交，不要本地 reset 后强推公共分支。

### 6.4 git revert 安全反转提交

**使用场景**：公共分支上的错误提交，需要撤销但保留历史。

**核心命令：**

```bash
# 反转指定提交（生成一次新的提交，内容与原提交相反）
git revert abc123
```

### 6.5 reset 与 revert 场景区分

| 维度 | reset | revert |
| --- | --- | --- |
| 历史 | 删除提交，修改历史 | 新增提交，不修改历史 |
| 哈希 | 后续提交哈希改变 | 不影响原有提交 |
| 公共分支 | 禁止使用 | 推荐使用 |
| 未推送的本地提交 | 推荐使用 | 没必要 |

**一句话总结：本地未推送用 reset，远程公共分支用 revert。**

### 6.6 修改提交信息

```bash
# 修改最近一次提交注释
git commit --amend -m "新的注释"

# 修改历史提交注释（交互式变基）
git rebase -i HEAD~3
# 把要修改的提交前的 pick 改成 reword，保存后依次修改
```

---

## 七、Tag 版本标签管理

### 7.1 适用场景

版本发布时打标签，对应一个确定的提交版本，方便回溯、发布。

### 7.2 核心命令

```bash
# 创建轻量标签
git tag v1.0.0

# 创建附注标签（推荐，带信息）
git tag -a v1.0.0 -m "正式发布1.0.0版本"

# 给指定提交打标签
git tag -a v1.0.0 abc123 -m "发布版本"

# 查看所有标签
git tag

# 查看标签详情
git show v1.0.0

# 删除本地标签
git tag -d v1.0.0

# 推送单个标签到远程
git push origin v1.0.0

# 推送所有标签
git push origin --tags

# 删除远程标签
git push origin -d v1.0.0
```

**最佳实践：**

- 版本号遵循语义化版本：主版本.次版本.修订号（如 v2.1.3）。
- 发布版本用附注标签，记录发布信息。

---

## 八、高阶实用操作

### 8.1 git cherry-pick 挑选提交

**使用场景**：把某个分支的某一次提交，单独合并到当前分支，不需要合并整个分支。

```bash
# 挑选指定提交
git cherry-pick abc123

# 挑选多个提交
git cherry-pick abc123 def456

# 挑选一段提交区间
git cherry-pick abc123..def456
```

### 8.2 git clean 清理未跟踪文件

**使用场景**：清理工作区中未被 Git 跟踪的文件，比如编译产物、临时文件。

```bash
# 查看会删除哪些文件（不实际删除）
git clean -n

# 删除未跟踪文件
git clean -f

# 删除未跟踪文件和目录
git clean -fd

# 连忽略的文件也删除
git clean -fdx
```

> 
> 先执行 `-n` 预览，确认无误再执行 `-f` 实际删除。

### 8.3 .gitignore 忽略文件

**使用场景**：指定不需要 Git 管理的文件，比如 node_modules、日志、本地配置。

**配置规则：**

```text
# 忽略单个文件
config.local

# 忽略目录
node_modules/
dist/

# 忽略一类文件
*.log
*.tmp

# 取反（不忽略）
!important.log

# 根目录下的某个文件
/build
```

**全局忽略配置：**

```bash
# 设置全局忽略文件
git config --global core.excludesfile ~/.gitignore_global
```

**避坑注意点：**

- 已经被 Git 跟踪的文件，再加入 `.gitignore` 无效，需要先 `git rm --cached 文件名` 取消跟踪。

---

## 九、冲突解决实战教程

### 9.1 冲突产生原因

两个分支修改了同一文件的同一行，Git 无法自动判断保留哪个，就会产生冲突，需要人工介入。

### 9.2 完整解决步骤

1. **执行合并/变基后出现冲突提示**
2. **查看冲突文件**：`git status` 红色标注 both modified 的文件
3. **打开文件手动解决**：

   ```text
   <<<<<<< HEAD
   当前分支的代码
   =======
   合并进来的代码
   >>>>>>> 分支名
   ```

   删除分隔符，保留最终正确的代码。

4. **解决完所有冲突后，加入暂存区**：

   ```bash
   git add .
   ```
5. **完成合并/变基**
   - merge 场景：`git commit`
   - rebase 场景：`git rebase --continue`

### 9.3 冲突保留规则

- 功能合并：保留两个功能的完整实现
- 同一行逻辑修改：和相关开发者确认后决定
- 不确定的冲突：不要擅自删除对方代码，沟通后再处理

### 9.4 多人协作避坑技巧

- 频繁拉取主分支更新，减少大冲突
- 按模块分工，尽量避免多人同时改同一文件
- 小步提交，避免一次提交几千行代码
- 合并前先自测，确保代码能正常运行

---

## 十、高频报错与解决方案

### 1. push 被拒绝（rejected）

**原因**：远程分支比本地新，有其他人推送了更新。
**解决：** 先 `git pull` 拉取合并，解决冲突后再 push。

### 2. 拉取冲突

**原因**：本地修改和远程修改冲突。
**解决：** 先 stash 暂存本地修改，pull 后再 pop stash，解决冲突。

### 3. 分支偏离（diverged）

**原因**：本地和远程历史不一致，通常是本地 rebase 了已推送的分支。
**解决：**

- 确认本地是对的：`git push -f` 强制推送（仅个人分支）
- 以远程为准：`git reset --hard origin/分支名`

### 4. 权限报错 Permission denied

**原因**：SSH 密钥未配置或没有仓库权限。
**解决：**

- 检查 SSH 密钥配置
- 确认仓库地址正确
- 确认账号有该仓库的读写权限

### 5. 超时/连接失败

**原因**：网络问题、远程地址错误、代理配置问题。
**解决：**

- 检查网络连通性
- 切换 HTTPS/SSH 地址
- 检查 Git 代理配置

---

## 十一、团队 Git 规范与最佳实践

### 11.1 分支命名规范

- 主分支：`main` / `master`（稳定可发布）
- 开发分支：`dev`（集成分支）
- 功能分支：`feature/xxx`
- 修复分支：`fix/xxx`
- 热修复分支：`hotfix/xxx`
- 发布分支：`release/xxx`

### 11.2 Commit 注释规范

统一使用 Conventional Commits 格式，便于自动生成 changelog、版本管理。

### 11.3 标准开发工作流

1. 从主分支拉取最新代码
2. 创建对应类型分支开发
3. 开发完成，自测通过
4. 拉取主分支最新代码，合并/变基同步
5. 解决冲突，提交
6. 推送远程，发起合并请求
7. Code Review 通过后合并入主分支

### 11.4 高危禁止操作

- 禁止在公共主分支执行 rebase
- 禁止强制推送公共分支
- 禁止直接在 main 分支提交代码
- 禁止提交构建产物、依赖包、密钥配置

### 11.5 日常高效习惯

- 小步提交，每个提交一个独立功能点
- 提交前先 diff 检查代码，避免误提交
- 每天至少拉取一次主分支更新
- 定期清理已合并的功能分支

---

## 附录一：Git 高频命令极简速查表

| 分类 | 命令 | 作用 |
| --- | --- | --- |
| 初始化 | `git init` | 初始化仓库 |
|  | `git clone 地址` | 克隆仓库 |
| 配置 | `git config --global user.name/email` | 全局配置 |
| 基础 | `git status` | 查看状态 |
|  | `git add .` | 添加全部到暂存区 |
|  | `git commit -m "注释"` | 提交 |
|  | `git log --oneline` | 查看日志 |
|  | `git diff` | 查看差异 |
| 分支 | `git branch` | 查看分支 |
|  | `git checkout -b 分支名` | 创建并切换 |
|  | `git merge 分支名` | 合并分支 |
|  | `git rebase 分支名` | 变基 |
| 远程 | `git remote add origin 地址` | 关联远程 |
|  | `git pull` | 拉取合并 |
|  | `git push` | 推送 |
| 暂存 | `git stash` | 暂存 |
|  | `git stash pop` | 恢复暂存 |
| 回滚 | `git restore 文件名` | 撤销工作区 |
|  | `git reset --hard HEAD~1` | 硬回滚 |
|  | `git revert 提交号` | 反转提交 |
| 标签 | `git tag v1.0.0` | 打标签 |
|  | `git push --tags` | 推送所有标签 |

---

## 附录二：新手高频踩坑误区汇总

1. **直接在 main 分支开发**：所有修改都在主分支，版本混乱，回滚困难。
2. **一次提交一大堆代码**：几十个功能混在一起，出问题无法定位。
3. **提交注释乱写**：`更新`、`修改`、`修复`，完全看不出改了什么。
4. **乱用 git reset --hard**：没备份就执行，代码直接丢失。
5. **公共分支 rebase**：修改公共历史，团队所有人都要处理冲突。
6. **强制推送主分支**：覆盖别人代码，造成团队事故。
7. **不配置 .gitignore**：把 node_modules、编译产物都提交了，仓库臃肿。
8. **冲突随便删代码**：不看内容直接删对方代码，导致功能丢失。
9. **长期不更新主分支**：最后合并时巨型冲突，难以解决。
10. **密码密钥提交到 Git**：安全隐患，泄露后后果严重。

---

## 附录三：个人/团队通用 Git 标准开发工作流

### 个人开发工作流

1. 拉取主分支最新代码
2. 创建功能分支开发
3. 小步提交，及时保存进度
4. 开发完成同步主分支更新
5. 自测验证，推送远程
6. 合并回主分支，删除功能分支

### 团队协作工作流（Git Flow 简化版）

1. `main`：生产环境分支，保护分支，只能通过合并请求合入
2. `dev`：开发集成分支，所有功能合并到这里
3. 开发人员从 `dev` 拉取，创建 `feature/xxx` 开发
4. 开发完成提 MR/PR 到 `dev`，Code Review 后合并
5. 发布版本从 `dev` 切 `release/xxx` 分支
6. 生产问题从 `main` 切 `hotfix/xxx` 修复，合并回 main 和 dev

---

> 
> 本文覆盖了 90% 以上日常开发场景的 Git 操作，从入门命令到团队规范，建议收藏备用。遇到不确定的操作，先查清楚再执行，尤其是涉及修改历史、强制推送的命令，谨慎操作。
