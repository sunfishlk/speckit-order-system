# Research: 员工点餐小程序技术调研

**Feature**: 员工点餐小程序  
**Date**: 2025-11-03  
**Purpose**: 解决技术选型和最佳实践问题

## 技术栈决策

### 1. Vite + Vanilla JS 前端方案

**Decision**: 使用 Vite 5.x 作为构建工具，纯 Vanilla JavaScript（无框架）

**Rationale**:
- **Vite 优势**：极快的冷启动（<1s）和热更新（<100ms），原生 ESM 支持
- **最小依赖**：符合用户要求"minimal number of libraries"，减少打包体积（预估 <100KB gzipped）
- **学习成本低**：新入职前端无需学习 React/Vue，直接使用 HTML/CSS/JS 基础知识
- **性能优越**：无虚拟 DOM 开销，直接操作原生 DOM，首屏渲染 <1s

**Alternatives Considered**:
- ❌ **React + Vite**：增加 40KB+ 运行时，违反最小依赖原则
- ❌ **Vue 3 + Vite**：虽然轻量（23KB），但仍引入额外学习曲线
- ❌ **Next.js/Nuxt**：过度工程化，不适合简单内部工具

**Best Practices**:
```javascript
// 使用 ES6 模块组织代码
// components/MenuList.js
export class MenuList {
  constructor(container) {
    this.container = container;
  }
  
  render(items) {
    this.container.innerHTML = items.map(item => `
      <div class="menu-item" data-id="${item.id}">
        <h3>${item.name}</h3>
        <p>¥${item.price.toFixed(2)}</p>
      </div>
    `).join('');
  }
}
```

---

### 2. Node.js + Express 后端方案

**Decision**: Node.js 18+ LTS + Express 4.x 框架

**Rationale**:
- **技术统一**：前后端都使用 JavaScript，降低团队技能要求
- **Express 成熟稳定**：14 年历史，npm 周下载量 3000万+，社区资源丰富
- **轻量级**：Express 核心仅 200KB，符合最小依赖原则
- **中间件生态**：丰富的中间件支持（CORS、日志、错误处理）

**Alternatives Considered**:
- ❌ **Fastify**：虽然性能更高（2x faster），但生态不如 Express 成熟
- ❌ **Koa**：需要额外学习 async/await 中间件模式，增加复杂度
- ❌ **NestJS**：TypeScript + 装饰器语法，学习曲线陡峭

**API Design Pattern**:
```javascript
// RESTful API 设计
GET    /api/menu             # 获取今日菜品
POST   /api/cart             # 更新购物车（本地存储）
POST   /api/orders           # 提交订单
GET    /api/orders           # 查询订单（行政端）
GET    /api/orders/export    # 导出 Excel
POST   /api/auth/wechat      # 企业微信 OAuth2 回调
```

---

### 3. SQLite 数据库方案

**Decision**: SQLite 3.x + better-sqlite3 驱动

**Rationale**:
- **无需部署**：单文件数据库（orders.db），无需 MySQL/PostgreSQL 服务器
- **性能优异**：本地读写延迟 <1ms，足够 100-500 员工规模
- **ACID 事务**：保证订单数据一致性（关键财务数据）
- **better-sqlite3 优势**：同步 API（更简单），性能优于 node-sqlite3（3-5x faster）

**Alternatives Considered**:
- ❌ **PostgreSQL**：过度工程化，需要额外部署和维护成本
- ❌ **MongoDB**：NoSQL 不适合财务数据（需要事务和关联查询）
- ❌ **MySQL**：需要独立服务器，增加运维负担

**Schema Design**:
```sql
-- 员工表
CREATE TABLE employees (
  id TEXT PRIMARY KEY,          -- 企业微信 userId
  name TEXT NOT NULL,
  department TEXT,
  created_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- 菜品表
CREATE TABLE menu_items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  category TEXT NOT NULL,       -- 中餐/轻食/饮品
  price INTEGER NOT NULL,        -- 单位：分（避免浮点数精度问题）
  available_date TEXT NOT NULL,  -- YYYY-MM-DD
  price_locked_at TEXT,          -- 价格锁定时间（HH:MM）
  created_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- 订单表
CREATE TABLE orders (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  employee_id TEXT NOT NULL,
  total_price INTEGER NOT NULL,  -- 单位：分
  status TEXT DEFAULT 'submitted', -- submitted/completed/cancelled
  submitted_at INTEGER DEFAULT (strftime('%s', 'now')),
  FOREIGN KEY (employee_id) REFERENCES employees(id)
);

-- 订单明细表
CREATE TABLE order_items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  order_id INTEGER NOT NULL,
  menu_item_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL,
  price_snapshot INTEGER NOT NULL, -- 下单时价格快照
  FOREIGN KEY (order_id) REFERENCES orders(id),
  FOREIGN KEY (menu_item_id) REFERENCES menu_items(id)
);

-- 索引优化查询性能
CREATE INDEX idx_orders_employee ON orders(employee_id);
CREATE INDEX idx_orders_date ON orders(submitted_at);
CREATE INDEX idx_menu_date ON menu_items(available_date);
```

---

### 4. 企业微信 OAuth2 认证

**Decision**: 企业微信网页授权 + JWT Token

**Rationale**:
- **零注册成本**：员工使用企业微信账号登录，无需额外注册
- **自动获取信息**：通过 OAuth2 自动获取姓名、部门、员工 ID
- **安全性**：企业微信提供 access_token + refresh_token 机制
- **JWT 会话管理**：无需服务器端 session 存储，适合无状态 API

**Alternatives Considered**:
- ❌ **手机号验证码**：需要短信服务费用（0.05元/条）和手机号管理
- ❌ **员工工号+密码**：需要密码管理和找回机制，增加复杂度
- ❌ **LDAP/AD**：需要企业 IT 部门配置，部署门槛高

**Auth Flow**:
```
1. 用户访问 /login → 重定向到企业微信授权页
   https://open.weixin.qq.com/connect/oauth2/authorize?
   appid=CORPID&redirect_uri=CALLBACK_URL&response_type=code

2. 用户同意授权 → 企业微信回调 /api/auth/wechat?code=CODE

3. 后端用 code 换取 access_token
   POST https://qyapi.weixin.qq.com/cgi-bin/gettoken

4. 用 access_token 获取用户信息
   GET https://qyapi.weixin.qq.com/cgi-bin/user/getuserinfo

5. 生成 JWT token 返回前端
   { userId, name, department, exp: 7天 }

6. 前端存储 JWT 到 localStorage，后续请求带上 Authorization: Bearer <JWT>
```

---

### 5. 购物车本地存储方案

**Decision**: localStorage + 定期同步到后端（可选）

**Rationale**:
- **离线优先**：用户可以在网络不稳定时继续选餐（PWA 特性）
- **减少服务器压力**：购物车频繁操作（增删改）不消耗后端资源
- **数据不丢失**：浏览器关闭后购物车内容保留，用户体验更好
- **简化实现**：无需后端 API 维护购物车状态

**Implementation**:
```javascript
// services/cart-storage.js
export class CartStorage {
  constructor() {
    this.key = 'order-food-cart';
  }
  
  get() {
    try {
      return JSON.parse(localStorage.getItem(this.key)) || [];
    } catch (e) {
      console.error('Failed to parse cart:', e);
      return [];
    }
  }
  
  set(cart) {
    localStorage.setItem(this.key, JSON.stringify(cart));
  }
  
  clear() {
    localStorage.removeItem(this.key);
  }
  
  // 计算总价（单位：元）
  getTotal() {
    const cart = this.get();
    return cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
  }
}
```

---

### 6. Excel 导出方案

**Decision**: 使用 `xlsx` 库（SheetJS）服务端生成

**Rationale**:
- **行业标准**：npm 周下载量 300万+，兼容 Excel 2007+ 格式
- **服务端生成**：避免前端内存压力（订单量大时）
- **格式灵活**：支持多工作表、样式、公式等高级功能
- **无浏览器兼容问题**：服务端统一生成，前端只需下载

**Alternatives Considered**:
- ❌ **CSV 格式**：无法支持多工作表和复杂格式，用户体验差
- ❌ **前端生成**：订单量大时（100+）可能导致浏览器卡顿
- ❌ **Google Sheets API**：需要外部服务依赖，增加复杂度

**Export API Implementation**:
```javascript
// services/export.js
import XLSX from 'xlsx';

export async function exportOrdersToExcel(date) {
  // 1. 查询订单数据（按菜品汇总 + 员工明细）
  const orders = await db.query(`
    SELECT o.id, e.name, e.department, 
           mi.name as menu_name, oi.quantity, oi.price_snapshot
    FROM orders o
    JOIN employees e ON o.employee_id = e.id
    JOIN order_items oi ON oi.order_id = o.id
    JOIN menu_items mi ON oi.menu_item_id = mi.id
    WHERE date(o.submitted_at, 'unixepoch') = ?
  `, [date]);
  
  // 2. 按菜品汇总
  const summary = groupBy(orders, 'menu_name').map(([name, items]) => ({
    菜品: name,
    数量: items.reduce((sum, i) => sum + i.quantity, 0),
    金额: items.reduce((sum, i) => sum + i.quantity * i.price_snapshot, 0) / 100
  }));
  
  // 3. 生成 Excel（两个工作表）
  const wb = XLSX.utils.book_new();
  const ws1 = XLSX.utils.json_to_sheet(summary);
  const ws2 = XLSX.utils.json_to_sheet(orders.map(o => ({
    订单号: o.id,
    员工: o.name,
    部门: o.department,
    菜品: o.menu_name,
    数量: o.quantity,
    单价: o.price_snapshot / 100,
    小计: (o.quantity * o.price_snapshot) / 100
  })));
  
  XLSX.utils.book_append_sheet(wb, ws1, '汇总');
  XLSX.utils.book_append_sheet(wb, ws2, '明细');
  
  // 4. 返回 Buffer
  return XLSX.write(wb, { type: 'buffer', bookType: 'xlsx' });
}
```

---

### 7. 价格锁定机制

**Decision**: 数据库 + Node-cron 定时任务

**Rationale**:
- **准确性**：服务端控制价格锁定时间，避免客户端时间不准
- **自动化**：定时任务每日自动执行，无需人工干预
- **可追溯**：价格锁定时间记录在数据库，便于审计

**Implementation**:
```javascript
// utils/price-lock.js
import cron from 'node-cron';
import db from '../db/index.js';

// 每日上午 10:00 执行价格锁定
cron.schedule('0 10 * * *', async () => {
  const today = new Date().toISOString().split('T')[0]; // YYYY-MM-DD
  
  await db.run(`
    UPDATE menu_items 
    SET price_locked_at = '10:00'
    WHERE available_date = ? AND price_locked_at IS NULL
  `, [today]);
  
  console.log(`[Price Lock] Locked prices for ${today} at 10:00`);
});

// 检查价格是否已锁定
export function isPriceLocked(availableDate) {
  const result = db.get(`
    SELECT price_locked_at FROM menu_items 
    WHERE available_date = ? LIMIT 1
  `, [availableDate]);
  
  return result && result.price_locked_at !== null;
}
```

---

### 8. 测试策略

**Decision**: Vitest (单元测试) + Playwright (E2E 测试)

**Rationale**:
- **Vitest**：Vite 官方测试工具，配置零成本，速度快（比 Jest 快 10x）
- **Playwright**：支持 Chrome/Safari/Firefox，模拟真实用户操作
- **覆盖率目标**：单元测试 >70%，E2E 测试覆盖 P1/P2 用户故事

**Test Structure**:
```javascript
// backend/tests/unit/order.test.js
import { describe, it, expect } from 'vitest';
import { createOrder } from '../../src/services/order.js';

describe('Order Service', () => {
  it('should calculate total price correctly', () => {
    const cart = [
      { id: 1, price: 25.00, quantity: 2 },
      { id: 2, price: 15.00, quantity: 1 }
    ];
    const total = calculateTotal(cart);
    expect(total).toBe(65.00);
  });
  
  it('should prevent empty cart submission', async () => {
    await expect(createOrder('user1', [])).rejects.toThrow('购物车为空');
  });
});

// frontend/tests/e2e/order-flow.spec.js
import { test, expect } from '@playwright/test';

test('员工完成点餐流程', async ({ page }) => {
  // 1. 打开首页
  await page.goto('/');
  
  // 2. 浏览菜品
  await expect(page.locator('.category-tab')).toHaveCount(3);
  
  // 3. 加入购物车
  await page.locator('[data-id="1"] .add-to-cart-btn').click();
  await expect(page.locator('.cart-badge')).toHaveText('1');
  
  // 4. 打开购物车
  await page.locator('.cart-button').click();
  await expect(page.locator('.cart-item')).toHaveCount(1);
  
  // 5. 提交订单
  await page.locator('.submit-btn').click();
  await expect(page.locator('.toast')).toHaveText('订单已提交！✅');
});
```

---

## 性能优化策略

### 1. 前端优化
- **代码分割**：按路由拆分（员工端 / 行政端），减少首屏加载
- **懒加载**：图片使用 `loading="lazy"` 属性
- **缓存策略**：菜品数据缓存 1 小时（Cache-Control: max-age=3600）
- **PWA**：Service Worker 缓存静态资源，支持离线访问

### 2. 后端优化
- **数据库索引**：在 employee_id、submitted_at、available_date 字段上建索引
- **查询优化**：使用 JOIN 而非 N+1 查询，减少数据库往返
- **连接池**：better-sqlite3 默认单连接，通过队列管理并发请求

### 3. 部署优化
- **Gzip 压缩**：Express 启用 compression 中间件，减少传输体积 70%
- **静态资源 CDN**：Vite 构建产物部署到 CDN（可选）
- **数据库备份**：每日定时备份 orders.db 文件

---

## 安全考虑

### 1. 认证安全
- **JWT 有效期**：7 天，到期后自动跳转登录页
- **HTTPS Only**：生产环境强制 HTTPS，防止 token 泄露
- **CORS 配置**：仅允许内网域名访问 API

### 2. 数据安全
- **SQL 注入防护**：使用参数化查询（better-sqlite3 自动处理）
- **XSS 防护**：前端使用 textContent 而非 innerHTML 插入用户输入
- **CSRF 防护**：API 使用 JWT 而非 Cookie，天然防 CSRF

### 3. 业务安全
- **订单幂等性**：检查同一天是否已提交订单，防止重复提交
- **价格篡改防护**：订单提交时后端验证价格，不信任前端数据
- **权限控制**：行政后台接口验证用户角色（admin role）

---

## 部署方案

### 开发环境
```bash
# 前端开发服务器（热更新）
cd frontend && npm run dev  # http://localhost:5173

# 后端开发服务器（nodemon 自动重启）
cd backend && npm run dev   # http://localhost:3000
```

### 生产环境
```bash
# 1. 构建前端
cd frontend && npm run build  # 输出到 dist/

# 2. 启动后端（提供静态文件服务 + API）
cd backend && npm start       # 监听 0.0.0.0:3000

# 3. 使用 PM2 进程管理（可选）
pm2 start backend/src/server.js --name order-food
```

### Docker 部署（可选）
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY backend ./backend
COPY frontend/dist ./backend/public
RUN cd backend && npm ci --only=production
EXPOSE 3000
CMD ["node", "backend/src/server.js"]
```

---

## 风险缓解

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 企业微信认证配置失败 | 无法登录 | 提供开发环境 Mock 登录，生产环境前完成配置测试 |
| SQLite 文件损坏 | 数据丢失 | 每日自动备份到云存储（阿里云 OSS/腾讯云 COS） |
| 并发订单冲突 | 数据不一致 | 使用事务（BEGIN TRANSACTION）保证原子性 |
| 菜品价格未锁定 | 结算混乱 | 启动时检查定时任务状态，发送告警通知 |
| 浏览器兼容性问题 | 部分用户无法使用 | 明确支持版本（Chrome 90+），提供升级提示 |

---

## 下一步行动

✅ **Phase 0 完成**：技术选型和最佳实践已确定  
➡️ **Phase 1 开始**：生成数据模型、API 契约、快速开始指南
