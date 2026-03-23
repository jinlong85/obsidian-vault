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
- 两台设备的 Obsidian 都在 **Windows系统** 运行
- 通过 Obsidian Git 插件自动同步
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

### Obsidian Vault

| 设备 | 路径 | remote URL | 认证方式 |
|------|------|------------|----------|
| MSI台式机 | `C:\Users\JINLONG\obsidian-vault` (Windows) | `git@github.com:jinlong85/obsidian-vault.git` | SSH |
| YOGA Pro16笔记本 | `C:\Users\JINLONG\obsidian-vault` (Windows) | `git@github.com:jinlong85/obsidian-vault.git` | SSH |

**重要**：Obsidian 安装在 **Windows系统**，不是WSL2。通过 Obsidian Git 插件同步 MD 文件。

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

**Windows SSH配置** (`C:\Users\JINLONG\.ssh\config`)：
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    AddKeysToAgent yes
```

**Windows SSH密钥位置**：`C:\Users\JINLONG\.ssh\id_ed25519`

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

### 3. 配置 SSH Key（用于 CC_TEST 和 claude-config）

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

### 6. Obsidian Vault

#### 6.1 安装 Obsidian 和 Clone vault

1. 在 Windows 安装 Obsidian：https://obsidian.md/download
2. Clone vault 到 Windows：
```powershell
cd C:\Users\JINLONG
git clone git@github.com:jinlong85/obsidian-vault.git
```

#### 6.2 安装 Obsidian Git 插件

1. 下载插件文件到 vault 的 `.obsidian\plugins\obsidian-git\` 目录：
   - manifest.json
   - main.js
   - styles.css

2. 启动 Obsidian，打开 vault

3. 启用插件：设置 → 社区插件 → 启用 Obsidian Git

#### 6.3 配置 SSH（重要！）

在 `C:\Users\JINLONG\.ssh\config` 添加：
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    AddKeysToAgent yes
```

从WSL2复制私钥：
```bash
cp ~/.ssh/id_ed25519 /mnt/c/Users/JINLONG/.ssh/
```

#### 6.4 配置插件

修改 `data.json`：
```json
{
  "autoSaveInterval": 1,
  "autoPushInterval": 2,
  "autoPullInterval": 2,
  "autoPullOnBoot": true,
  "autoBackupAfterFileChange": true,
  "differentIntervalCommitAndPush": false
}
```

---

## 七、GitHub 仓库列表

| 仓库 | 地址 | 用途 |
|------|------|------|
| CC_TEST 项目 | `git@github.com:jinlong85/cc_test.git` | Node.js 脚本项目 |
| Claude Code 配置 | `git@github.com:jinlong85/claude-config.git` | Claude Code 设置同步 |
| Obsidian Vault | `git@github.com:jinlong85/obsidian-vault.git` | 知识库同步（Windows系统） |

---

## 八、验证命令

### 检查同步状态
```bash
bash ~/sync-all.sh
```

### 检查 GitHub SSH 连接
```bash
ssh -T git@github.com
```

### 检查 vault remote
```bash
git remote -v
# 应该显示 https://github.com/jinlong85/obsidian-vault.git
```

---

## 九、常见问题与解决方案

### 问题 1: Windows Git SSH 认证失败

**现象**：`git@github.com: Permission denied (publickey)`

**原因**：Windows Git 没有配置 SSH key

**解决方案**：WSL 用 SSH，Windows vault 用 HTTPS

---

### 问题 2: Obsidian Git 插件搜不到

**解决方案**：手动下载插件文件到 vault 的 `.obsidian\plugins\obsidian-git\` 目录

---

### 问题 3: Obsidian Git 插件 SSH 认证失败

**现象**：`Host key verification failed` 或 `Permission denied (publickey)`

**原因**：Windows 缺少 SSH 私钥或未配置 SSH config

**解决方案**：
```bash
# 1. 从WSL2复制私钥到Windows
cp ~/.ssh/id_ed25519 /mnt/c/Users/JINLONG/.ssh/

# 2. 创建SSH config
# 在 C:\Users\JINLONG\.ssh\config 中添加：
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    AddKeysToAgent yes
```

---

### 问题 4: PowerShell Git 代理问题

**现象**：`Failed to connect to github.com port 443`

**解决方案**：
```powershell
git config --global http.proxy http://127.0.0.1:9999
git config --global https.proxy http://127.0.0.1:9999
```

---

### 问题 5: Git 用户信息未配置

**现象**：`Author identity unknown`

**解决方案**：
```bash
git config --global user.name "jinlong85"
git config --global user.email "jinlong85@users.noreply.github.com"
```

---

### 问题 6: autoPullOnBoot 为 false

**现象**：重启 Obsidian 后不自动拉取

**解决方案**：确保 `data.json` 中 `"autoPullOnBoot": true`

---

## 十、重要规则

1. **Obsidian 在 Windows 系统运行** - 不是WSL2，通过 Obsidian Git 插件同步
2. **所有仓库统一使用 SSH** - 包括 Obsidian Vault（需要配置 SSH key）
3. **先配置 Git 用户信息** - `git config --global user.name/email`
4. **Windows 需要配置 SSH config** - `C:\Users\JINLONG\.ssh\config`
5. **第三台电脑部署按第六节流程** - 确保不遗漏步骤
