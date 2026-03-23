# 记忆 - Claude Code + Obsidian 双设备同步系统

---

## 一、设备信息

| 设备 | 角色 | CC_TEST 路径 | Obsidian Vault 路径 |
|------|------|--------------|---------------------|
| MSI 台式机 | 宿舍主力机 | `/home/jinlong/CC_TEST` | `C:\Users\JINLONG\obsidian-vault` |
| YOGA Pro16 笔记本 | 办公室 | `/home/jinlong/cc_test` | `C:\Users\JINLONG\obsidian-vault` |

---

## 二、同步架构

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│  MSI 台式机      │ ← ── → │    GitHub       │ ← ── → │  YOGA Pro16 笔记本 │
│  (Windows)      │  push   │   (中转)        │  push   │  (Windows)      │
│  Obsidian       │  pull   │                 │  pull   │  Obsidian       │
└─────────────────┘         └─────────────────┘         └─────────────────┘
```

**双向同步机制**：
- Obsidian 在 **Windows系统** 运行，通过 Git 插件同步
- 通过 GitHub 作为中转站

---

## 三、仓库信息

### CC_TEST 项目

| 设备 | 路径 | remote URL | 认证方式 |
|------|------|------------|----------|
| 台式机 | `/home/jinlong/CC_TEST` | `git@github.com:jinlong85/cc_test.git` | SSH |
| 笔记本 | `/home/jinlong/cc_test` | `git@github.com:jinlong85/cc_test.git` | SSH |

### Claude Code 配置

| 设备 | 路径 | remote URL | 认证方式 |
|------|------|------------|----------|
| 台式机 | `~/.claude-config` | `git@github.com:jinlong85/claude-config.git` | SSH |
| 笔记本 | `~/.claude-config` | `git@github.com:jinlong85/claude-config.git` | SSH |

### Obsidian Vault（Windows）

| 设备 | 路径 | remote URL | 认证方式 |
|------|------|------------|----------|
| 台式机 | `C:\Users\JINLONG\obsidian-vault` | `git@github.com:jinlong85/obsidian-vault.git` | SSH |
| 笔记本 | `C:\Users\JINLONG\obsidian-vault` | `git@github.com:jinlong85/obsidian-vault.git` | SSH |

**Obsidian 运行在 Windows**，通过 Obsidian Git 插件同步，不是 WSL2。

---

## 四、同步脚本

### 脚本位置
- 台式机：`~/sync-all.sh`
- 笔记本：`~/sync-all.sh`

### 脚本内容（双向同步）
```bash
#!/bin/bash
# CC_TEST 项目双向同步
cd ~/CC_TEST  # 台式机
# cd ~/cc_test  # 笔记本
git pull origin main 2>/dev/null
if [ -n "$(git status --porcelain)" ]; then
    git add .
    git commit -m "auto backup: $(date '+%Y-%m-%d %H:%M:%S')"
    git push origin main 2>/dev/null
fi

# Claude Config 双向同步
cd ~/.claude-config
git pull origin master 2>/dev/null
if git diff --quiet settings.json 2>/dev/null && git diff --cached --quiet settings.json 2>/dev/null; then
    :
else
    git add settings.json
    git commit -m "auto backup: $(date '+%Y-%m-%d %H:%M:%S')"
    git push origin master 2>/dev/null
fi
```

### 定时任务
```bash
*/5 * * * * /home/jinlong/sync-all.sh
```

---

## 五、Obsidian Git 插件配置

### 插件位置
```
C:\Users\JINLONG\obsidian-vault\.obsidian\plugins\obsidian-git\
├── main.js
├── manifest.json
└── styles.css
```

### 推荐配置（直接覆盖 data.json）
```json
{
  "commitMessage": "vault backup: {{date}}",
  "autoCommitMessage": "vault backup: {{date}}",
  "commitDateFormat": "YYYY-MM-DD HH:mm:ss",
  "autoSaveInterval": 1,
  "autoPushInterval": 2,
  "autoPullInterval": 2,
  "autoPullOnBoot": true,
  "disablePush": false,
  "pullBeforePush": true,
  "disablePopups": false,
  "listChangedFilesInMessageBody": false,
  "showStatusBar": true,
  "updateSubmodules": false,
  "syncMethod": "merge",
  "customMessageOnAutoBackup": false,
  "autoBackupAfterFileChange": true,
  "treeStructure": false,
  "refreshSourceControl": true,
  "basePath": "",
  "differentIntervalCommitAndPush": false,
  "changedFilesInStatusBar": false,
  "username": "",
  "showedMobileNotice": true,
  "refreshSourceControlTimer": 7000,
  "showBranchStatusBar": true
}
```

### Windows SSH 配置
`C:\Users\JINLONG\.ssh\config`：
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    AddKeysToAgent yes
```

从 WSL2 复制私钥：
```bash
cp ~/.ssh/id_ed25519 /mnt/c/Users/JINLONG/.ssh/
```

---

## 六、第三台电脑部署流程

### 1. WSL2 + VS Code + Claude Code 环境

1. 安装 WSL2
2. 安装 VS Code + Remote - WSL 插件
3. 安装 Claude Code

### 2. 配置 Git

```bash
git config --global user.name "jinlong85"
git config --global user.email "jinlong85@users.noreply.github.com"
```

### 3. 配置 SSH Key

```bash
ssh-keygen -t ed25519 -C "jinlong85" -f ~/.ssh/id_ed25519 -N ""
cat ~/.ssh/id_ed25519.pub
# 添加到 GitHub
```

### 4. Clone 项目

```bash
git clone git@github.com:jinlong85/cc_test.git ~/cc_test
git clone git@github.com:jinlong85/claude-config.git ~/.claude-config
cp ~/.claude-config/settings.json ~/.claude/settings.json
```

### 5. 同步脚本

```bash
cat > ~/sync-all.sh << 'EOF'
#!/bin/bash
cd ~/cc_test
git pull origin main 2>/dev/null
if [ -n "$(git status --porcelain)" ]; then
    git add .
    git commit -m "auto backup: $(date '+%Y-%m-%d %H:%M:%S')"
    git push origin main 2>/dev/null
fi

cd ~/.claude-config
git pull origin master 2>/dev/null
if git diff --quiet settings.json 2>/dev/null && git diff --cached --quiet settings.json 2>/dev/null; then
    :
else
    git add settings.json
    git commit -m "auto backup: $(date '+%Y-%m-%d %H:%M:%S')"
    git push origin master 2>/dev/null
fi
EOF
chmod +x ~/sync-all.sh
(crontab -l 2>/dev/null; echo "*/5 * * * * /home/jinlong/sync-all.sh") | crontab -
```

### 6. Obsidian Vault（Windows）

#### 6.1 安装 Obsidian 和 Clone vault

```powershell
cd C:\Users\JINLONG
git clone git@github.com:jinlong85/obsidian-vault.git
```

#### 6.2 安装 Obsidian Git 插件

手动下载插件到 `.obsidian\plugins\obsidian-git\`：
- manifest.json
- main.js
- styles.css

#### 6.3 配置 SSH

从 WSL2 复制私钥到 Windows：
```bash
cp ~/.ssh/id_ed25519 /mnt/c/Users/JINLONG/.ssh/
```

创建 `C:\Users\JINLONG\.ssh\config`：
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    AddKeysToAgent yes
```

---

## 七、GitHub 仓库列表

| 仓库 | 地址 | 用途 |
|------|------|------|
| CC_TEST 项目 | `git@github.com:jinlong85/cc_test.git` | Node.js 脚本项目 |
| Claude Code 配置 | `git@github.com:jinlong85/claude-config.git` | Claude Code 设置同步 |
| Obsidian Vault | `git@github.com:jinlong85/obsidian-vault.git` | 知识库同步 |

---

## 八、验证命令

### 检查 SSH 连接
```bash
ssh -T git@github.com
```

### 检查 vault remote
```bash
git remote -v
# 应显示：git@github.com:jinlong85/obsidian-vault.git
```

### 检查同步状态
```bash
bash ~/sync-all.sh
```

---

## 九、重要规则

1. **所有仓库统一使用 SSH**
2. **Obsidian 在 Windows 运行**，通过 Git 插件同步，不是 WSL2
3. **先配置 Git 用户信息**：`git config --global user.name/email`
4. **Windows 需要 SSH config**：否则 Obsidian Git 插件认证失败
5. **第三台电脑部署按第六节流程**

---

## 十、常见问题

### 分支分叉（diverged）

**现象**：`git push` 被拒绝，提示 `! [rejected] main -> main (fetch first)`

**解决**（保留本地版本）：
```bash
git reset --hard origin/main && git push --force
```

### 文件显示 modified 但无内容变化

通常是换行符（CRLF/LF）或文件权限（100644/100755）差异，无实质内容不同。

### SSH 认证失败

1. 检查私钥是否存在：`ls ~/.ssh/id_ed25519`
2. 检查 SSH config 是否正确配置
3. 验证连接：`ssh -T git@github.com`
