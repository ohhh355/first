###yangcheng
### 1. 安装git

在 **PowerShell** 里执行：

```
# 1) 安装 Git（官方包）
winget install --id Git.Git -e

# 2) 关闭并重开 VS Code / 终端，然后检查版本
git --version
```

> 如果提示找不到 `winget`，可以改用 **Chocolatey**：
>
> ```
> choco install git -y
> ```
>
> 或者手动去“Git for Windows”安装程序，一路“下一步”，并确保选择“Add Git to PATH”。

安装向导里建议选项（有就勾上）：

- “**Use Git from the command line and also from 3rd-party software**”（把 Git 加入 PATH）
- 默认编辑器选 VS Code（可选）
- “**Git Credential Manager**”（方便推送到 GitHub）
- 行尾换行：**Checkout Windows-style, commit Unix-style**（推荐）

**常见情况与快速排错**

- 仍然提示“无法识别 git”：
   关闭并**重新打开** VS Code/PowerShell；确认 PATH 中有
   `C:\Program Files\Git\cmd`。你也可以在 PowerShell 里检查：

  ```
  where.exe git
  ```

  找不到就说明没进 PATH，重新运行安装程序并勾选“Add Git to PATH”。

- OneDrive 里冲突多：把项目移到 `C:\dev\...` 之类的本地目录会更省心；或在 OneDrive 设置里排除该文件夹同步。



**一次性自检清单**

```
git --version
git remote -v
git rev-parse --abbrev-ref HEAD    # 应为 main（或你期望的分支）
git log --oneline -n 3
```



-------------------------

### 2. 初始化仓库（只做一次）

在项目根目录打开终端（VS Code 内置终端就行）：

```
# 初始化本地仓库
git init

# 设置你的身份（只需设置一次，全局生效）
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"

# 建立 .gitignore（见文末模板）
echo ".venv/
__pycache__/
*.pyc
*.log
logs/
dist/
build/
*.qt*.cache
" >> .gitignore

# 首次提交
git add .
git commit -m "chore: initial commit"
```

> 说明：`.gitignore` 会让 Git 忽略无关文件（如编译产物、缓存、虚拟环境、日志）。

------

### 3. 规范行尾与清理已跟踪的无关文件

#### 3.1 行尾（CRLF/LF）规范

```
# 建议提交为 LF，检出按平台转换
git config core.autocrlf true

# 用 .gitattributes 固化
@"
* text=auto eol=lf
*.bat eol=crlf
*.cmd eol=crlf
*.ps1 eol=crlf
"@ | Out-File -Encoding utf8 .gitattributes

git add .gitattributes
git commit -m "chore: add .gitattributes for line endings"
```

#### 3.2 停止追踪缓存/日志（你之前提交过一次）

```
git rm -r --cached .
git add .
git commit -m "chore: apply .gitignore; stop tracking caches and logs"
git status --ignored
```





### 4.  每次写代码后的标准提交流程（创建了代码之后，后续更改只需执行第四步）

无论是在主分支（`main`）还是功能分支，习惯如下：

```
# 查看变更
git status

# 看具体改了什么（彩色 diff）
git diff

# 选择性加入需要提交的文件
git add main_window.py control_manager.py

# 提交（写清楚本次做了什么）
git commit -m "feat(control): add ControlManager mock and UI tick refresh"

# 推送到 GitHub
git push
```

> VS Code 里也能做同样操作：左侧「源代码管理」面板
>
> 1. 在「更改」里点每个文件右侧的「+」可暂存（stage）
> 2. 上方输入提交说明，点对勾（✓）即可提交
> 3. 点击任意文件可直接查看**两栏对比 diff**





你已经有代码了，直接用 `git apply` 会把**一次性的改动**写入工作区，然后你再提交，这样历史里就会清楚记录“引入 control_manager / on_tick / 清空函数”等变更。

```
git apply vlbi_control_system.patch
git add .
git commit -m "feat: integrate control_manager, tick update, clear_status_fields"
```

------

### 

### 5. 查看历史与回溯（最常用）

- **查看项目历史**（命令行）：

  ```
  git log --oneline --graph --decorate
  ```

- **查看某个文件的历史**（命令行）：

  ```
  git log -- main_window.py
  git show HEAD~1:main_window.py   # 看上一个版本的文件内容
  ```

- **VS Code 查看历史**：

  - 打开文件 → 右键 → **Timeline（时间线）** 里可看历史版本
  - 左侧「源代码管理」或装个 **GitLens** 插件：更直观的历史/责任人/行注释

- **回滚**（安全做法是建新提交回滚）：

  ```
  git revert <某个commit的哈希>
  ```

  或者临时看看旧版本：

  ```
  git checkout <commit哈希> -- main_window.py
  ```

------

### 6. 分支与合并（推荐小步快跑）

建议主分支只放稳定版本；开发用功能分支：

```
# 新建并切到功能分支
git checkout -b feat/control-manager

# 正常开发... add/commit

# 合并回主分支
git checkout main
git merge --no-ff feat/control-manager -m "merge: feat/control-manager"
```

> 在 VS Code 底部状态栏也能快速切换/创建分支。

------

### 7.  绑定 GitHub 并推送

#### 7.1 在 GitHub 新建**空**仓库

- 名称：`vlbi_control_software`

- 不要勾选 README / .gitignore / License（保持空仓库）

- 你的远端地址形如：

  ```
  https://github.com/df24-blip/vlbi_control_software.git
  ```

#### 7.2 设置远端并推送

```
# 重命名分支为 main（推荐）
git branch -M main

# 绑定远端（把占位符换成你的真实仓库地址）
git remote add origin https://github.com/df24-blip/vlbi_control_software.git
git remote -v    # 自检：应显示正确地址

# 首次推送（会弹 GitHub 登录框）
git push -u origin main
```

> 如果远端不是空仓库（你手滑加了 README），解决：(更新README)
>
> ```
> git pull --rebase origin main
> git push -u origin main
> ```
>
> 若远端有初始化提交（README等）且历史不相关：
>
> ```
> git pull --rebase origin main --allow-unrelated-histories
> ```
>
> 

​	

------

### 9. 提交信息怎么写更清晰（建议规范）

- 前缀类型：`feat` 新功能、`fix` 修复、`chore` 杂项、`refactor` 重构、`docs` 文档、`test` 测试
- 可选作用域：`(ui)`、`(control)`、`(xml)`
- 简洁描述：一句话说清做了什么

例：

- `feat(control): add ControlManager mock and timer tick`
- `fix(ui): clear status fields after stop to avoid stale values`
- `test(xml): add minimal loadXML case`

------

### 10.  适合你项目的 `.gitignore`（可复制）

`.gitignore` 是 Git 项目里一个**专门的配置文件**，用来告诉 Git：

> “哪些文件我不想被纳入版本控制。”

```
# Python
__pycache__/
*.py[cod]
*.pyo
*.so
*.egg-info/
.venv/
venv/
.env

# Logs
*.log
logs/

# Build/Dist
build/
dist/
*.spec

# Qt / PyQt (有些平台会生成临时缓存)
*.qrc.dep
*.qtcreator*
*.user

# OS
.DS_Store
Thumbs.db
```

------

### 11.  一些小技巧（可选）

- **查看某行是谁改的**：在 VS Code 里开 GitLens，行尾会显示作者与提交；或命令行 `git blame main_window.py -L 100,150`
- **为一个稳定点打标签**：`git tag -a v0.1.0 -m "first demo"; git push origin v0.1.0`
- **临时保存未完成的改动**：`git stash; git switch 其他分支; git stash pop`