# Data Model: 员工点餐小程序

**Feature**: 员工点餐小程序  
**Date**: 2025-11-03  
**Storage**: SQLite 3.x

## 实体关系图（ER Diagram）

```
┌─────────────┐       ┌──────────────┐       ┌─────────────┐
│  employees  │──────<│    orders    │>──────│ menu_items  │
│             │ 1    N│              │N    1 │             │
│ id (PK)     │       │ id (PK)      │       │ id (PK)     │
│ name        │       │ employee_id  │       │ name        │
│ department  │       │ total_price  │       │ category    │
└─────────────┘       │ status       │       │ price       │
                      │ submitted_at │       └─────────────┘
                      └──────────────┘              │
                             │                      │
                             │ 1                  N │
                             │                      │
                      ┌──────▼──────────────────────┘
                      │    order_items              │
                      │                             │
                      │ id (PK)                     │
                      │ order_id (FK)               │
                      │ menu_item_id (FK)           │
                      │ quantity                    │
                      │ price_snapshot              │
                      └─────────────────────────────┘
```

---

## 核心实体

### 1. Employee（员工）

**用途**：存储员工基本信息，通过企业微信 OAuth2 自动同步

| 字段名 | 类型 | 约束 | 说明 | 示例 |
|--------|------|------|------|------|
| id | TEXT | PRIMARY KEY | 企业微信 userId（唯一标识） | "zhangsan" |
| name | TEXT | NOT NULL | 员工姓名 | "张三" |
| department | TEXT | NULL | 所属部门 | "产品部" |
| created_at | INTEGER | DEFAULT now | 创建时间（Unix 时间戳） | 1699000000 |

**验证规则**（Validation Rules）:
- `id`: 必填，长度 1-64 字符，仅包含字母数字下划线
- `name`: 必填，长度 2-50 字符
- `department`: 可选，长度 ≤100 字符

**状态转换**：无（员工信息静态，不涉及状态流转）

**索引**:
- PRIMARY KEY on `id`（自动创建）

---

### 2. MenuItem（菜品）

**用途**：存储可选菜品信息，支持按日期和类别管理

| 字段名 | 类型 | 约束 | 说明 | 示例 |
|--------|------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 菜品唯一标识 | 1 |
| name | TEXT | NOT NULL | 菜品名称 | "宫保鸡丁套餐" |
| category | TEXT | NOT NULL | 菜品类别 | "中餐" |
| price | INTEGER | NOT NULL | 单价（单位：分） | 2500 (¥25.00) |
| available_date | TEXT | NOT NULL | 供应日期（YYYY-MM-DD） | "2025-11-03" |
| price_locked_at | TEXT | NULL | 价格锁定时间（HH:MM） | "10:00" |
| created_at | INTEGER | DEFAULT now | 创建时间（Unix 时间戳） | 1699000000 |

**验证规则**:
- `name`: 必填，长度 2-100 字符
- `category`: 必填，枚举值 ["中餐", "轻食", "饮品"]
- `price`: 必填，> 0，≤ 1000000（最高 ¥10000）
- `available_date`: 必填，格式 YYYY-MM-DD，必须 ≥ 今日
- `price_locked_at`: 可选，格式 HH:MM（24小时制）

**业务规则**:
1. **价格锁定机制**：每日上午 10:00 自动锁定当日价格（`price_locked_at = '10:00'`）
2. **价格快照**：订单提交时保存 `price_snapshot`，防止价格变动影响已下订单
3. **日期过滤**：前端仅展示 `available_date = 今日` 的菜品

**索引**:
- `CREATE INDEX idx_menu_date ON menu_items(available_date);`（加速按日期查询）
- `CREATE INDEX idx_menu_category ON menu_items(category);`（加速按类别过滤）

---

### 3. Order（订单）

**用途**：存储员工提交的订单主记录

| 字段名 | 类型 | 约束 | 说明 | 示例 |
|--------|------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 订单唯一标识 | 1 |
| employee_id | TEXT | NOT NULL, FK | 下单员工 ID | "zhangsan" |
| total_price | INTEGER | NOT NULL | 订单总价（单位：分） | 6500 (¥65.00) |
| status | TEXT | DEFAULT 'submitted' | 订单状态 | "submitted" |
| submitted_at | INTEGER | DEFAULT now | 提交时间（Unix 时间戳） | 1699000000 |

**验证规则**:
- `employee_id`: 必填，必须存在于 `employees` 表
- `total_price`: 必填，> 0，= SUM(order_items.quantity * order_items.price_snapshot)
- `status`: 枚举值 ["submitted", "completed", "cancelled"]

**业务规则**:
1. **每日一单限制**：同一 `employee_id` 在同一天（`date(submitted_at) = 今日`）只能提交 1 个订单
2. **价格一致性**：`total_price` 必须等于所有 `order_items` 的小计之和（后端验证）
3. **状态流转**：
   - `submitted` → `completed`（行政确认配送完成）
   - `submitted` → `cancelled`（员工/行政取消订单）

**索引**:
- `CREATE INDEX idx_orders_employee ON orders(employee_id);`（加速按员工查询）
- `CREATE INDEX idx_orders_date ON orders(submitted_at);`（加速按日期查询）

---

### 4. OrderItem（订单明细）

**用途**：存储订单中的具体菜品和数量

| 字段名 | 类型 | 约束 | 说明 | 示例 |
|--------|------|------|------|------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | 明细唯一标识 | 1 |
| order_id | INTEGER | NOT NULL, FK | 所属订单 ID | 1 |
| menu_item_id | INTEGER | NOT NULL, FK | 菜品 ID | 1 |
| quantity | INTEGER | NOT NULL | 购买数量 | 2 |
| price_snapshot | INTEGER | NOT NULL | 下单时单价快照（单位：分） | 2500 |

**验证规则**:
- `order_id`: 必填，必须存在于 `orders` 表
- `menu_item_id`: 必填，必须存在于 `menu_items` 表
- `quantity`: 必填，> 0，≤ 100（单个菜品最多 100 份）
- `price_snapshot`: 必填，> 0，必须等于下单时 `menu_items.price`

**业务规则**:
1. **价格快照**：`price_snapshot` 记录下单时的价格，即使后续菜品价格变动也不影响已下订单
2. **关联删除**：删除订单时，级联删除所有 `order_items`（`ON DELETE CASCADE`）

**索引**:
- `CREATE INDEX idx_order_items_order ON order_items(order_id);`（加速查询订单明细）

---

## 数据完整性约束

### 外键约束（Foreign Keys）
```sql
-- orders 表外键
ALTER TABLE orders ADD CONSTRAINT fk_orders_employee
  FOREIGN KEY (employee_id) REFERENCES employees(id)
  ON DELETE RESTRICT;  -- 禁止删除有订单的员工

-- order_items 表外键
ALTER TABLE order_items ADD CONSTRAINT fk_order_items_order
  FOREIGN KEY (order_id) REFERENCES orders(id)
  ON DELETE CASCADE;  -- 删除订单时自动删除明细

ALTER TABLE order_items ADD CONSTRAINT fk_order_items_menu
  FOREIGN KEY (menu_item_id) REFERENCES menu_items(id)
  ON DELETE RESTRICT;  -- 禁止删除已被订购的菜品
```

### 唯一性约束（Unique Constraints）
```sql
-- 员工每日只能提交一个订单
CREATE UNIQUE INDEX idx_orders_employee_date 
  ON orders(employee_id, date(submitted_at, 'unixepoch'));
```

### 检查约束（Check Constraints）
```sql
-- 价格必须 > 0
ALTER TABLE menu_items ADD CONSTRAINT chk_menu_price_positive
  CHECK (price > 0);

ALTER TABLE orders ADD CONSTRAINT chk_order_total_positive
  CHECK (total_price > 0);

-- 数量必须 > 0
ALTER TABLE order_items ADD CONSTRAINT chk_quantity_positive
  CHECK (quantity > 0 AND quantity <= 100);

-- 订单状态枚举
ALTER TABLE orders ADD CONSTRAINT chk_order_status
  CHECK (status IN ('submitted', 'completed', 'cancelled'));

-- 菜品类别枚举
ALTER TABLE menu_items ADD CONSTRAINT chk_menu_category
  CHECK (category IN ('中餐', '轻食', '饮品'));
```

---

## 查询示例（Common Queries）

### 1. 查询今日可选菜品（按类别分组）
```sql
SELECT category, id, name, price, price_locked_at
FROM menu_items
WHERE available_date = DATE('now')
ORDER BY category, id;
```

### 2. 查询员工今日是否已下单
```sql
SELECT COUNT(*) as order_count
FROM orders
WHERE employee_id = ? 
  AND date(submitted_at, 'unixepoch') = DATE('now');
```

### 3. 查询指定日期的所有订单（行政后台）
```sql
SELECT 
  o.id as order_id,
  e.name as employee_name,
  e.department,
  o.total_price,
  o.submitted_at,
  GROUP_CONCAT(mi.name || ' x' || oi.quantity) as items
FROM orders o
JOIN employees e ON o.employee_id = e.id
JOIN order_items oi ON oi.order_id = o.id
JOIN menu_items mi ON oi.menu_item_id = mi.id
WHERE date(o.submitted_at, 'unixepoch') = ?
GROUP BY o.id
ORDER BY o.submitted_at DESC;
```

### 4. 按菜品汇总统计（用于导出 Excel）
```sql
SELECT 
  mi.category,
  mi.name,
  SUM(oi.quantity) as total_quantity,
  SUM(oi.quantity * oi.price_snapshot) as total_revenue
FROM order_items oi
JOIN menu_items mi ON oi.menu_item_id = mi.id
JOIN orders o ON oi.order_id = o.id
WHERE date(o.submitted_at, 'unixepoch') = ?
GROUP BY mi.id
ORDER BY mi.category, total_quantity DESC;
```

---

## 数据迁移（Migrations）

### V1: 初始化表结构
```sql
-- backend/src/db/migrations/001_initial_schema.sql

-- 员工表
CREATE TABLE IF NOT EXISTS employees (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  department TEXT,
  created_at INTEGER DEFAULT (strftime('%s', 'now'))
);

-- 菜品表
CREATE TABLE IF NOT EXISTS menu_items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  category TEXT NOT NULL CHECK (category IN ('中餐', '轻食', '饮品')),
  price INTEGER NOT NULL CHECK (price > 0),
  available_date TEXT NOT NULL,
  price_locked_at TEXT,
  created_at INTEGER DEFAULT (strftime('%s', 'now'))
);

CREATE INDEX idx_menu_date ON menu_items(available_date);
CREATE INDEX idx_menu_category ON menu_items(category);

-- 订单表
CREATE TABLE IF NOT EXISTS orders (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  employee_id TEXT NOT NULL,
  total_price INTEGER NOT NULL CHECK (total_price > 0),
  status TEXT DEFAULT 'submitted' CHECK (status IN ('submitted', 'completed', 'cancelled')),
  submitted_at INTEGER DEFAULT (strftime('%s', 'now')),
  FOREIGN KEY (employee_id) REFERENCES employees(id)
);

CREATE INDEX idx_orders_employee ON orders(employee_id);
CREATE INDEX idx_orders_date ON orders(submitted_at);
CREATE UNIQUE INDEX idx_orders_employee_date ON orders(employee_id, date(submitted_at, 'unixepoch'));

-- 订单明细表
CREATE TABLE IF NOT EXISTS order_items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  order_id INTEGER NOT NULL,
  menu_item_id INTEGER NOT NULL,
  quantity INTEGER NOT NULL CHECK (quantity > 0 AND quantity <= 100),
  price_snapshot INTEGER NOT NULL CHECK (price_snapshot > 0),
  FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
  FOREIGN KEY (menu_item_id) REFERENCES menu_items(id)
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
```

---

## 数据种子（Seed Data）

### 开发环境测试数据
```sql
-- backend/src/db/seeds/dev_data.sql

-- 插入测试员工
INSERT INTO employees (id, name, department) VALUES
  ('zhangsan', '张三', '产品部'),
  ('lisi', '李四', '技术部'),
  ('wangwu', '王五', '行政部');

-- 插入今日菜品
INSERT INTO menu_items (name, category, price, available_date) VALUES
  ('宫保鸡丁套餐', '中餐', 2500, DATE('now')),
  ('红烧肉套餐', '中餐', 2800, DATE('now')),
  ('麻婆豆腐套餐', '中餐', 2200, DATE('now')),
  ('鱼香肉丝套餐', '中餐', 2400, DATE('now')),
  ('凯撒沙拉', '轻食', 3200, DATE('now')),
  ('三明治组合', '轻食', 2800, DATE('now')),
  ('牛油果鸡胸沙拉', '轻食', 3500, DATE('now')),
  ('美式咖啡', '饮品', 1500, DATE('now')),
  ('鲜榨橙汁', '饮品', 1800, DATE('now')),
  ('柠檬茶', '饮品', 1200, DATE('now')),
  ('冰镇可乐', '饮品', 800, DATE('now'));
```

---

## 数据备份策略

### 自动备份脚本
```bash
#!/bin/bash
# backend/scripts/backup-db.sh

# 备份目录
BACKUP_DIR="./data/backups"
mkdir -p "$BACKUP_DIR"

# 生成备份文件名（带时间戳）
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/orders_$TIMESTAMP.db"

# 复制数据库文件
cp ./data/orders.db "$BACKUP_FILE"

# 压缩备份
gzip "$BACKUP_FILE"

# 删除 7 天前的备份
find "$BACKUP_DIR" -name "orders_*.db.gz" -mtime +7 -delete

echo "Backup completed: ${BACKUP_FILE}.gz"
```

### 定时任务（Cron）
```bash
# 每天凌晨 2:00 自动备份
0 2 * * * /path/to/backend/scripts/backup-db.sh
```

---

## 性能优化建议

### 1. 查询优化
- ✅ 已创建索引：`employee_id`, `submitted_at`, `available_date`
- ✅ 使用 `date()` 函数时建立函数索引
- ⚠️ 避免 `SELECT *`，明确列出需要的字段
- ⚠️ 使用 `EXPLAIN QUERY PLAN` 分析慢查询

### 2. 连接池配置
```javascript
// better-sqlite3 是同步库，使用队列管理并发
import Queue from 'queue';

const dbQueue = new Queue({ concurrency: 1, autostart: true });

export function query(sql, params) {
  return new Promise((resolve, reject) => {
    dbQueue.push(() => {
      try {
        const result = db.prepare(sql).all(params);
        resolve(result);
      } catch (err) {
        reject(err);
      }
    });
  });
}
```

### 3. 数据归档
```sql
-- 每月归档上月订单到历史表（可选）
CREATE TABLE IF NOT EXISTS orders_archive AS SELECT * FROM orders WHERE 1=0;

INSERT INTO orders_archive
SELECT * FROM orders
WHERE date(submitted_at, 'unixepoch') < DATE('now', 'start of month');
```

---

## 下一步
✅ **Phase 1: 数据模型完成**  
➡️ **接下来**：生成 API 契约（OpenAPI 规范）
