# Implementation Plan: 员工点餐小程序

**Branch**: `001-employee-meal-ordering` | **Date**: 2025-11-03 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/001-employee-meal-ordering/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

员工点餐小程序，解决行政专员统一订餐的统计和结算痛点。
员工通过 Web 界面浏览菜品、加入购物车、提交订单；行政查看汇总清单并导出 Excel。
技术方案：Vite + Vanilla JS 前端，Node.js 后端，SQLite 数据库，企业微信 OAuth2 认证。

## Technical Context

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

**Language/Version**: Node.js 18+ (后端), HTML5/CSS3/ES2020 (前端)  
**Primary Dependencies**: Vite 5.x (构建工具), better-sqlite3 (数据库驱动), express (Web 框架)  
**Storage**: SQLite 3.x (本地文件数据库)  
**Testing**: Vitest (单元测试), Playwright (E2E 测试)  
**Target Platform**: Web (Chrome 90+, Safari 14+, WeChat 内嵌浏览器)
**Project Type**: web (前后端分离 Web 应用)  
**Performance Goals**: 首屏加载 <2s, API 响应 <200ms, 支持 100 并发用户  
**Constraints**: 离线优先（PWA），数据库文件 <100MB，购物车本地存储  
**Scale/Scope**: 100-500 员工，12+ 菜品，日均 100+ 订单

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### I. 受众优先 (Audience-First Writing)
✅ **通过** - 本文档使用短句，专业词汇已释义（如 PWA = 渐进式 Web 应用，OAuth2 = 开放授权协议）

### II. 需求三要素 (Requirement Triad)
✅ **通过** - 每个功能点包含是什么/为什么/怎么验收（见 spec.md User Stories）

### III. 禁止空话 (Zero-Fluff Policy)
✅ **通过** - 所有指标可量化（如 <2s 加载，100 并发，200ms 响应）

### IV. 流程最短化 (Minimal Flow Principle)
✅ **通过** - 点餐流程仅 3 步（浏览 → 购物车 → 提交），符合 ≤3 步要求

### V. 隐私与合规 (Privacy & Compliance - NON-NEGOTIABLE)
✅ **通过** - 企业微信 OAuth2 认证，订单数据本地加密存储，符合 GDPR 最小权限原则

### 评估结论
所有原则检查通过，无需豁免。可进入 Phase 0 研究阶段。

## Project Structure

### Documentation (this feature)

```text
specs/[###-feature]/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
backend/
├── src/
│   ├── db/
│   │   ├── schema.sql       # SQLite 数据库表结构
│   │   └── migrations/       # 数据库迁移脚本
│   ├── models/
│   │   ├── employee.js       # 员工模型
│   │   ├── menu-item.js      # 菜品模型
│   │   └── order.js          # 订单模型
│   ├── services/
│   │   ├── auth.js           # 企业微信 OAuth2 认证
│   │   ├── menu.js           # 菜品管理服务
│   │   ├── order.js          # 订单处理服务
│   │   └── export.js         # Excel 导出服务
│   ├── api/
│   │   ├── routes/           # API 路由定义
│   │   └── middleware/       # 中间件（认证、错误处理）
│   ├── utils/
│   │   ├── price-lock.js     # 价格锁定逻辑
│   │   └── logger.js         # 日志工具
│   └── server.js             # 服务器入口
├── tests/
│   ├── unit/                 # 单元测试
│   └── integration/          # 集成测试
└── package.json

frontend/
├── src/
│   ├── components/
│   │   ├── MenuList.js       # 菜品列表组件
│   │   ├── Cart.js           # 购物车组件
│   │   └── AdminPanel.js     # 管理后台组件
│   ├── pages/
│   │   ├── index.html        # 员工点餐页面
│   │   └── admin.html        # 行政后台页面
│   ├── services/
│   │   ├── api.js            # API 调用封装
│   │   └── cart-storage.js   # 购物车本地存储
│   ├── styles/
│   │   └── main.css          # 全局样式
│   └── main.js               # 应用入口
├── tests/
│   └── e2e/                  # E2E 测试 (Playwright)
├── public/
│   ├── manifest.json         # PWA 配置
│   └── service-worker.js     # 离线支持
├── vite.config.js
└── package.json

data/
└── orders.db                 # SQLite 数据库文件
```

**Structure Decision**: 选择 Web 应用结构（Option 2），前后端分离。
- **frontend/**: Vite 构建的 SPA，Vanilla JS 实现，最小化依赖
- **backend/**: Express.js REST API，SQLite 数据持久化
- **data/**: SQLite 数据库文件存储目录

**理由**：
1. 前后端分离便于独立开发和部署
2. Vite 提供快速开发体验和高效构建
3. SQLite 无需额外数据库服务器，简化部署
4. Vanilla JS 减少学习成本，符合"最小依赖"要求

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
