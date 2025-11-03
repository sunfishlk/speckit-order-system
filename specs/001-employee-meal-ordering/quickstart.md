# Quick Start: 员工点餐小程序

**Feature**: 员工点餐小程序  
**Date**: 2025-11-03  
**Target Audience**: 新入职的前端/后端/QA 同学

本文档帮助你在 15 分钟内完成项目环境搭建和首次运行。

---

## 前置要求（Prerequisites）

在开始之前，请确保已安装以下工具：

| 工具 | 最低版本 | 检查命令 | 安装链接 |
|------|----------|----------|----------|
| Node.js | 18.x | `node --version` | https://nodejs.org |
| npm | 9.x | `npm --version` | (随 Node.js 自带) |
| Git | 2.x | `git --version` | https://git-scm.com |

---

## 快速开始（3 步）

### 1. 克隆项目并安装依赖

```bash
# 克隆仓库
git clone <repository-url>
cd order-food

# 切换到功能分支
git checkout 001-employee-meal-ordering

# 安装后端依赖
cd backend
npm install

# 安装前端依赖
cd ../frontend
npm install
```

**预计耗时**: 2-3 分钟

---

### 2. 初始化数据库

```bash
# 回到项目根目录
cd ..

# 创建数据目录
mkdir -p data

# 初始化数据库（创建表结构）
cd backend
node src/db/init.js

# 插入测试数据（开发环境）
node src/db/seed.js
```

**预计耗时**: 10 秒

**验证**：检查 `data/orders.db` 文件是否创建成功
```bash
ls -lh data/orders.db
# 应该看到约 20KB 的 SQLite 文件
```

---

### 3. 启动开发服务器

打开两个终端窗口（或使用 tmux/screen）：

**终端 1 - 启动后端**:
```bash
cd backend
npm run dev
# 输出: Server running on http://localhost:3000
```

**终端 2 - 启动前端**:
```bash
cd frontend
npm run dev
# 输出: Local: http://localhost:5173
```

**预计耗时**: 30 秒

---

## 验证安装（Smoke Test）

### 1. 访问前端页面
打开浏览器访问 `http://localhost:5173`

你应该看到：
- ✅ 员工点餐首页（显示今日菜品）
- ✅ 菜品分类标签（中餐、轻食、饮品）
- ✅ 购物车图标（右下角浮动按钮）

### 2. 测试核心功能

**测试点餐流程**:
1. 点击任意菜品的"加入购物车"按钮
2. 购物车图标显示数量角标（如 `1`）
3. 点击购物车图标，打开购物车弹窗
4. 验证菜品名称、数量、总价显示正确
5. 点击"提交订单"按钮
6. 看到"订单已提交！✅"提示

**测试管理后台**:
1. 点击右上角"管理后台"按钮
2. 查看订单统计（订单总数、营业额）
3. 浏览订单列表（员工姓名、菜品、金额）
4. 点击"导出订单清单"按钮，下载 CSV 文件

### 3. 验证 API 端点

使用 curl 或 Postman 测试后端 API：

```bash
# 获取今日菜品
curl http://localhost:3000/api/menu

# 预期返回 JSON:
# { "success": true, "data": { "date": "2025-11-03", "items": [...] } }
```

---

## 项目结构（Quick Tour）

```
order-food/
├── backend/                 # Node.js 后端
│   ├── src/
│   │   ├── db/             # 数据库相关（schema, migrations, seed）
│   │   ├── models/         # 数据模型（Employee, MenuItem, Order）
│   │   ├── services/       # 业务逻辑（auth, menu, order, export）
│   │   ├── api/            # API 路由和中间件
│   │   ├── utils/          # 工具函数（price-lock, logger）
│   │   └── server.js       # 服务器入口
│   ├── tests/              # 测试文件
│   └── package.json
│
├── frontend/               # Vite + Vanilla JS 前端
│   ├── src/
│   │   ├── components/     # UI 组件（MenuList, Cart, AdminPanel）
│   │   ├── pages/          # 页面（index.html, admin.html）
│   │   ├── services/       # API 调用和本地存储
│   │   ├── styles/         # 全局样式
│   │   └── main.js         # 应用入口
│   ├── public/             # 静态资源（PWA manifest）
│   ├── vite.config.js      # Vite 配置
│   └── package.json
│
├── data/                   # SQLite 数据库文件
│   └── orders.db
│
└── specs/                  # 需求和设计文档
    └── 001-employee-meal-ordering/
        ├── spec.md         # 功能规格说明
        ├── plan.md         # 实施计划
        ├── research.md     # 技术调研
        ├── data-model.md   # 数据模型
        ├── contracts/      # API 契约（OpenAPI）
        └── quickstart.md   # 本文档
```

---

## 常用命令（Cheat Sheet）

### 后端命令

```bash
# 开发模式（自动重启）
npm run dev

# 生产模式
npm start

# 运行测试
npm test

# 数据库迁移
npm run db:migrate

# 插入测试数据
npm run db:seed

# 备份数据库
npm run db:backup
```

### 前端命令

```bash
# 开发模式（热更新）
npm run dev

# 构建生产版本
npm run build

# 预览构建产物
npm run preview

# 运行 E2E 测试
npm run test:e2e

# 代码格式化
npm run lint
```

---

## 开发环境配置

### 1. 后端环境变量

创建 `backend/.env` 文件：

```bash
# 服务器配置
NODE_ENV=development
PORT=3000
HOST=0.0.0.0

# 数据库路径
DATABASE_PATH=../data/orders.db

# JWT 密钥（生产环境必须修改）
JWT_SECRET=your-secret-key-change-in-production
JWT_EXPIRES_IN=7d

# 企业微信配置（需要企业 IT 提供）
WECHAT_CORP_ID=your-corp-id
WECHAT_AGENT_ID=your-agent-id
WECHAT_SECRET=your-secret
WECHAT_CALLBACK_URL=http://localhost:3000/api/auth/wechat/callback

# 价格锁定时间（每日上午 10:00）
PRICE_LOCK_TIME=10:00

# 日志级别
LOG_LEVEL=info
```

### 2. 前端环境变量

创建 `frontend/.env` 文件：

```bash
# API 基础 URL
VITE_API_BASE_URL=http://localhost:3000/api

# 企业微信应用 ID（可选，用于直接跳转）
VITE_WECHAT_CORP_ID=your-corp-id
```

---

## 常见问题（FAQ）

### Q1: 后端启动失败，提示 "Error: ENOENT: no such file or directory, open '../data/orders.db'"

**原因**: 数据库文件未创建

**解决**:
```bash
cd backend
node src/db/init.js
```

---

### Q2: 前端无法连接后端 API，提示 "Network Error"

**原因**: 后端服务未启动或 CORS 配置问题

**解决**:
1. 检查后端是否运行: `curl http://localhost:3000/api/menu`
2. 检查 `backend/src/server.js` 中的 CORS 配置

---

### Q3: 企业微信登录失败，提示 "invalid code"

**原因**: 企业微信应用配置不正确

**解决**（开发环境）:
1. 使用 Mock 登录绕过企业微信认证（仅开发环境）
2. 访问 `http://localhost:5173/?mock=zhangsan` 直接登录为测试用户

**解决**（生产环境）:
1. 联系企业 IT 部门配置企业微信应用
2. 确认 `WECHAT_CORP_ID`, `WECHAT_SECRET` 等配置正确
3. 检查回调 URL 是否在企业微信后台白名单中

---

### Q4: 数据库文件损坏，提示 "database disk image is malformed"

**原因**: SQLite 文件损坏（断电、磁盘故障等）

**解决**:
1. 恢复备份: `cp data/backups/orders_YYYYMMDD.db.gz data/ && gunzip data/orders_*.db.gz`
2. 重新初始化: `rm data/orders.db && node backend/src/db/init.js && node backend/src/db/seed.js`

---

### Q5: 价格锁定定时任务不生效

**原因**: Node-cron 未正确启动

**解决**:
1. 检查 `backend/src/utils/price-lock.js` 是否在 `server.js` 中引入
2. 查看服务器日志: `grep "Price Lock" backend/logs/app.log`
3. 手动触发锁定: `curl -X POST http://localhost:3000/api/admin/lock-prices`

---

## 下一步（Next Steps）

✅ **环境搭建完成**！接下来可以：

1. **阅读需求文档**: `specs/001-employee-meal-ordering/spec.md`
2. **了解数据模型**: `specs/001-employee-mel-ordering/data-model.md`
3. **查看 API 文档**: `specs/001-employee-meal-ordering/contracts/openapi.yaml`
4. **开始开发任务**: `specs/001-employee-meal-ordering/tasks.md`（需先运行 `/speckit.tasks`）

---

## 技术支持

遇到问题？按优先级尝试以下方式：

1. **查看文档**: `specs/001-employee-meal-ordering/` 目录下的设计文档
2. **搜索日志**: `backend/logs/app.log` 和浏览器控制台
3. **提问团队**: 在团队群/Slack 提问，附上错误日志和重现步骤
4. **提交 Issue**: 在 Git 仓库创建 Issue，描述问题和环境信息

---

## 附录：IDE 配置建议

### VSCode 推荐扩展

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",       // ESLint 代码检查
    "esbenp.prettier-vscode",       // 代码格式化
    "bradlc.vscode-tailwindcss",    // CSS 智能提示
    "ms-vscode.vscode-typescript-next", // TypeScript 支持
    "ritwickdey.liveserver"         // 快速预览 HTML
  ]
}
```

### 推荐的调试配置

创建 `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Backend",
      "program": "${workspaceFolder}/backend/src/server.js",
      "cwd": "${workspaceFolder}/backend",
      "envFile": "${workspaceFolder}/backend/.env"
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Debug Frontend",
      "url": "http://localhost:5173",
      "webRoot": "${workspaceFolder}/frontend/src"
    }
  ]
}
```

---

**祝开发愉快！🚀**
