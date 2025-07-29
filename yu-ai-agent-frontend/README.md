# AI智能体应用平台前端

这是一个基于Vue3开发的AI智能体应用平台，包含三个核心应用：AI恋爱大师、AI超级智能体和小学必背古诗。

## 功能特点

- 💬 **AI恋爱大师**：智能情感顾问，为用户提供恋爱相关的建议和解答
- 🤖 **AI超级智能体**：全能型AI助手，解决各类专业问题
- 📚 **小学必背古诗**：75首经典古诗词，支持搜索、分类浏览和详细展示

## 技术栈

- Vue3
- Vue Router
- Axios
- SSE (Server-Sent Events)

## 开发说明

### 环境要求

- Node.js >= 16.0.0
- npm >= 7.0.0

### 安装依赖

```bash
npm install
```

### 启动开发服务器

```bash
npm run dev
```

### 构建项目

```bash
npm run build
```

## 部署说明

项目已配置多种部署方案，选择其中一种即可：

### 1. Vercel 部署（推荐）

1. 将代码推送到GitHub仓库
2. 访问 [vercel.com](https://vercel.com)
3. 导入GitHub仓库
4. 自动部署完成

### 2. GitHub Pages 部署

1. 将代码推送到GitHub仓库
2. 在仓库设置中启用GitHub Pages
3. 选择 `gh-pages` 分支作为源
4. GitHub Actions会自动构建和部署

### 3. Netlify 部署

1. 访问 [netlify.com](https://netlify.com)
2. 拖拽 `dist` 文件夹到部署区域
3. 或连接GitHub仓库自动部署

## 后端接口

项目依赖以下后端接口：

- `/api/ai/love_app/chat/sse` - AI恋爱大师聊天接口
- `/api/ai/manus/chat` - AI超级智能体聊天接口

后端服务默认运行在 `http://localhost:8123`

## 项目结构

```
src/
├── components/          # 公共组件
├── views/              # 页面组件
│   ├── Home.vue        # 首页
│   ├── LoveMaster.vue  # AI恋爱大师
│   ├── SuperAgent.vue  # AI超级智能体
│   └── Poetry.vue      # 小学必背古诗
├── router/             # 路由配置
└── api/                # API接口
```
