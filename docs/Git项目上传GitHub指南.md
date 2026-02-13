# Git 项目上传 GitHub 实战指南

## 核心流程（主干）

### 1. 准备工作
```bash
# 生成SSH密钥
ssh-keygen -t ed25519 -C "你的邮箱"
# 一路回车即可
```

### 2. 添加SSH密钥到GitHub
- 复制公钥：`cat ~/.ssh/id_ed25519.pub` （整行复制）
- 打开：https://github.com/settings/ssh/new
- 粘贴公钥，点击 Add SSH key

### 3. 初始化本地仓库
```bash
git init
git add .
git commit -m "Initial commit: 项目初始化"
```

### 4. 创建GitHub仓库
- 打开：https://github.com/new
- 输入仓库名
- **重要：不要勾选 "Add a README file"**

### 5. 推送代码
```bash
# 添加远程仓库
git remote add origin git@github.com:用户名/仓库名.git

# 切换到main分支（GitHub默认）
git branch -M main

# 推送
git push -u origin main
```

---

## 避坑指南

### 坑1：SSH 22端口被拒绝
**现象**：`ssh: connect to host github.com port 22: Connection refused`

**解决方案**：配置SSH使用443端口
```bash
# 创建/编辑 ~/.ssh/config
echo "Host github.com
    Hostname ssh.github.com
    Port 443
    User git" > ~/.ssh/config

# 测试连接
ssh -T -p 443 git@ssh.github.com
```

### 坑2：Git LFS 冲突
**现象**：`error: GH008: Your push referenced at least 1 unknown Git LFS object`

**原因**：
- `.gitattributes` 配置了 `*.pdf filter=lfs` 等LFS规则
- 文件已被LFS跟踪，但LFS对象丢失

**解决方案**：
```bash
# 方案1：移除LFS配置，重新提交
rm -rf .git
echo "* text=auto" > .gitattributes  # 不包含LFS配置
git init
git add .
git commit -m "Initial commit"
git push -u origin main
```

**预防措施**：
- 小文件（<100MB）不要用LFS
- 检查 `.gitattributes`，避免 `filter=lfs` 配置

### 坑3：分支名不匹配
**现象**：推送后GitHub上看不到代码

**原因**：本地用 `master`，GitHub默认 `main`

**解决**：`git branch -M main` 再推送

### 坑4：GitHub网页编辑后本地推送被拒绝 ⭐ 常见
**现象**：
```
Updates were rejected because the tip of your current branch is behind
failed to push some refs to ...
```

**原因**：
- 在GitHub网页上直接编辑了文件（如README.md）
- 网页修改产生了新提交，远程仓库版本领先本地
- 本地推送时版本冲突被拒绝

**解决方案**：

**方法1：先拉取再推送（推荐）**
```bash
# 拉取并合并远程更改
git pull origin main

# 如有冲突，解决后重新提交
# 然后再推送
git push origin main
```

**方法2：使用rebase（更清晰的提交历史）**
```bash
# 变基合并，保持线性历史
git pull --rebase origin main
git push origin main
```

**方法3：查看差异后手动合并**
```bash
# 先看看远程改了什么
git fetch origin
git log HEAD..origin/main  # 查看远程新增的提交

# 再决定如何合并
git merge origin/main
# 或 git rebase origin/main
```

⚠️ **警告**：绝对不要用 `git push --force`，会覆盖网页上的修改！

---

### 坑5：Windows 文件名冲突
**现象**：`fatal: unable to index file 'nul'`

**原因**：`nul` 是Windows保留设备名

**解决**：删除冲突文件
```bash
rm nul 2>/dev/null
del nul 2>/dev/null
```

---

## 最佳实践

### .gitignore 模板
```gitignore
# IDE配置
.idea/
.vscode/

# Python缓存
__pycache__/
*.pyc

# 虚拟环境
venv/
env/

# 日志和临时文件
*.log
*.tmp

# 系统文件
.DS_Store
Thumbs.db

# 密钥和敏感信息
.env
*.key

# 大文件（模型权重、输出图片）
*.safetensors
*.bin
output_*.png
```

### 提交信息规范
```bash
git commit -m "feat: 新增功能
fix: 修复bug
docs: 文档更新
refactor: 代码重构
style: 格式修改
test: 测试相关"
```

---

## 快速检查清单

- [ ] SSH密钥已生成并添加到GitHub
- [ ] `.gitignore` 已配置（避免提交垃圾文件）
- [ ] 分支名改为 `main`
- [ ] `.gitattributes` 中无LFS配置（除非确实需要）
- [ ] GitHub仓库创建时未勾选 "Add README"
- [ ] 网络环境能访问 GitHub（必要时配置代理或使用SSH 443端口）

---

## 常用命令速查

```bash
# 查看状态
git status

# 查看远程仓库
git remote -v

# 查看分支
git branch

# 查看提交历史
git log --oneline -5

# 修改最后一次提交
git add .
git commit --amend --no-edit

# 强制推送（慎用）
git push --force
```

---

## 自动化检查：避免推送冲突

### 创建推送前检查脚本

为了防止"GitHub网页编辑后本地推送被拒绝"的问题，可以创建一个Git Hook，在推送前自动检查远程是否有更新。

**Windows (PowerShell)** - 创建 `.git/hooks/pre-push`：
```powershell
# 文件路径：.git/hooks/pre-push
# 需要保存为无扩展名的文件

$remoteBranch = "origin/main"
$currentBranch = git rev-parse --abbrev-ref HEAD

if ($currentBranch -eq "main" -or $currentBranch -eq "master") {
    # 获取远程最新提交
    git fetch origin quiet 2>$null
    $localCommit = git rev-parse HEAD
    $remoteCommit = git rev-parse origin/$currentBranch 2>$null

    if ($localCommit -ne $remoteCommit -and $remoteCommit -ne $null) {
        $behind = git rev-list --count HEAD..origin/$currentBranch
        if ([int]$behind -gt 0) {
            Write-Host "⚠️  警告：你的本地分支落后远程 $behind 个提交" -ForegroundColor Yellow
            Write-Host "建议先运行：git pull origin $currentBranch" -ForegroundColor Cyan
            Write-Host ""
            $response = Read-Host "是否继续推送？(y/N)"
            if ($response -ne "y" -and $response -ne "Y") {
                exit 1
            }
        }
    }
}

exit 0
```

**跨平台方案（Bash脚本）** - 创建 `.git/hooks/pre-push`：
```bash
#!/bin/bash
# 文件路径：.git/hooks/pre-push

# 获取当前分支名
CURRENT_BRANCH=$(git rev-parse --abbrev-ref HEAD)

# 只检查 main/master 分支
if [[ "$CURRENT_BRANCH" == "main" || "$CURRENT_BRANCH" == "master" ]]; then
    # 静默获取远程更新
    git fetch origin quiet 2>/dev/null

    # 检查是否落后
    BEHIND=$(git rev-list --count HEAD..origin/$CURRENT_BRANCH 2>/dev/null)

    if [[ $BEHIND -gt 0 ]]; then
        echo "⚠️  警告：你的本地分支落后远程 $BEHIND 个提交"
        echo "建议先运行：git pull origin $CURRENT_BRANCH"
        echo ""
        read -p "是否继续推送？ " -n 1 -r
        echo
        if [[ ! $REPLY =~ ^[Yy]$ ]]; then
            exit 1
        fi
    fi
fi

exit 0
```

**安装步骤**：
1. 将上述脚本保存为 `.git/hooks/pre-push`（无扩展名）
2. 给脚本添加执行权限（Linux/Mac）：`chmod +x .git/hooks/pre-push`
3. 每次推送前会自动检查

---

## 推荐工作流程

### 方案A：纯本地编辑（推荐）
```bash
# 1. 本地修改代码
# 2. 提交
git add .
git commit -m "xxx"

# 3. 推送前先拉取（避免冲突）
git pull origin main

# 4. 推送
git push origin main
```

### 方案B：网页编辑 + 本地编辑（容易冲突）
如果你经常在GitHub网页上编辑：
```bash
# 每次开始工作前
git pull origin main  # 先同步远程更改

# 然后再修改本地代码
# ... 编辑代码 ...

# 推送时再次检查
git fetch origin
git log HEAD..origin/main  # 查看远程是否有新提交

git pull --rebase origin main  # 使用rebase保持历史清晰
git push origin main
```

### 方案C：团队协作最佳实践
```bash
# 1. 为每个任务创建新分支
git checkout -b feature/new-feature

# 2. 在分支上工作
git add .
git commit -m "feat: xxx"

# 3. 推送分支
git push -u origin feature/new-feature

# 4. 在GitHub上创建Pull Request
# 5. 代码审查后合并到main
# 6. 本地同步最新main
git checkout main
git pull origin main
```

**优势**：
- ✅ 不影响主分支
- ✅ 可以在网页上安全编辑README等
- ✅ 冲突隔离在PR中解决
- ✅ 符合团队协作规范

---

## 参考资源

- [GitHub Docs: Dealing with non-fast-forward errors](https://docs.github.com/en/get-started/using-git/dealing-with-non-fast-forward-errors)
- [Stack Overflow: Git Push Rejected](https://stackoverflow.com/questions/620253/git-push-rejected)
- [Medium: Resolving GitHub's Push Declined Due to Email Privacy](https://medium.com/@python-javascript-php-css/resolving-githubs-push-declined-due-to-email-privacy-restrictions-issue-c346a9cf1da0)
