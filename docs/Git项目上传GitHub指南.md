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

### 坑4：Windows 文件名冲突
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
