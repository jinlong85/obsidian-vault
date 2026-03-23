# CC_TEST 项目

## 基本信息

- **项目路径**: `/home/jinlong/CC_TEST`
- **类型**: Node.js 脚本工具集
- **创建时间**: 2024年

## 项目结构

```
CC_TEST/
├── av_merger/          # 音视频合并工具
├── claude-skills/      # Claude Code 技能目录
├── scripts/            # 脚本工具
├── weibo_photos/       # 微博图片下载
├── node_modules/       # npm 依赖
├── package.json        # 项目配置
└── weibo-downloader.js # 微博下载器
```

## 技术栈

- Node.js (ES6+)
- npm 包管理
- Chrome CDP (用于自动化)

## 代码规范

1. 使用 ES6+ 语法
2. 错误处理要完善，try-catch 不能少
3. async/await 优先于回调
4. 使用 console.error 输出错误日志

## 常用命令

```bash
# 安装依赖
npm install

# 运行脚本
node weibo-downloader.js
node test_chrome.js

# 下载 YouTube
./ytb-download.sh
```

## 注意事项

- WSL 环境下路径格式与 Windows 不同，注意兼容性
- API 请求注意代理配置
