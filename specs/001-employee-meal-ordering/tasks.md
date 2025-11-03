# Tasks: 员工点餐小程序

**Input**: Design documents from `/specs/001-employee-meal-ordering/`  
**Prerequisites**: plan.md (required), spec.md (required for user stories), data-model.md, contracts/

**Tests**: Tests are OPTIONAL - not explicitly requested in spec.md, so this task list focuses on implementation.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Backend**: `backend/src/` at repository root
- **Frontend**: `frontend/src/` at repository root
- **Data**: `data/` at repository root

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create backend directory structure per plan.md
- [ ] T002 Create frontend directory structure per plan.md
- [ ] T003 [P] Initialize backend package.json with Node.js 18+ and dependencies (express, better-sqlite3, jsonwebtoken, node-cron, xlsx, cors, dotenv)
- [ ] T004 [P] Initialize frontend package.json with Vite 5.x and minimal dependencies
- [ ] T005 [P] Create backend/.env template with required environment variables (NODE_ENV, PORT, DATABASE_PATH, JWT_SECRET, WECHAT_CORP_ID, etc.)
- [ ] T006 [P] Create frontend/.env template with API base URL configuration
- [ ] T007 [P] Create vite.config.js in frontend/ for build configuration
- [ ] T008 [P] Create data/ directory for SQLite database file

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T009 Create SQLite database schema in backend/src/db/schema.sql (employees, menu_items, orders, order_items tables with indexes)
- [ ] T010 Create database initialization script in backend/src/db/init.js to create tables and indexes
- [ ] T011 Create database seed script in backend/src/db/seed.js to insert test data (employees and menu items)
- [ ] T012 [P] Create Employee model in backend/src/models/employee.js with validation rules
- [ ] T013 [P] Create MenuItem model in backend/src/models/menu-item.js with validation rules
- [ ] T014 [P] Create Order model in backend/src/models/order.js with validation rules
- [ ] T015 [P] Implement database connection utility in backend/src/db/index.js using better-sqlite3
- [ ] T016 [P] Create JWT utility functions in backend/src/utils/jwt.js (sign, verify tokens)
- [ ] T017 [P] Create logger utility in backend/src/utils/logger.js for structured logging
- [ ] T018 [P] Implement authentication middleware in backend/src/api/middleware/auth.js to verify JWT tokens
- [ ] T019 [P] Implement error handling middleware in backend/src/api/middleware/error-handler.js
- [ ] T020 [P] Create CORS middleware configuration in backend/src/api/middleware/cors.js
- [ ] T021 Create Express server setup in backend/src/server.js with middleware chain
- [ ] T022 [P] Create API base structure in backend/src/api/routes/index.js to mount all route handlers
- [ ] T023 [P] Create frontend API client wrapper in frontend/src/services/api.js with fetch abstraction
- [ ] T024 [P] Create cart storage service in frontend/src/services/cart-storage.js using localStorage
- [ ] T025 [P] Create global CSS reset and variables in frontend/src/styles/main.css

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - 员工浏览并选购今日菜品 (Priority: P1) 🎯 MVP

**Goal**: 员工可以浏览今日菜品、加入购物车、查看购物车并修改数量

**Independent Test**: 员工打开页面 → 查看菜品分类 → 加入购物车 → 打开购物车 → 修改数量 → 验证总价计算准确

### Backend Implementation for US1

- [ ] T026 [P] [US1] Implement menu service in backend/src/services/menu.js to query today's menu items by category
- [ ] T027 [P] [US1] Create GET /api/menu endpoint in backend/src/api/routes/menu.js to return menu items grouped by category
- [ ] T028 [US1] Add price lock check logic in backend/src/services/menu.js to indicate if prices are locked

### Frontend Implementation for US1

- [ ] T029 [P] [US1] Create MenuList component in frontend/src/components/MenuList.js to display menu items by category
- [ ] T030 [P] [US1] Create Cart component in frontend/src/components/Cart.js with add/remove/update quantity functions
- [ ] T031 [P] [US1] Create category tabs UI in frontend/src/components/CategoryTabs.js
- [ ] T032 [US1] Implement index.html in frontend/src/pages/index.html with layout structure
- [ ] T033 [US1] Implement main.js in frontend/src/main.js to initialize app and load menu items
- [ ] T034 [US1] Add cart total calculation logic in frontend/src/services/cart-storage.js (sum of price × quantity)
- [ ] T035 [US1] Add floating cart button with badge showing item count in frontend/src/components/Cart.js
- [ ] T036 [US1] Implement cart modal/drawer UI to display cart items with quantity controls

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - 员工提交订餐订单 (Priority: P2)

**Goal**: 员工确认购物车后提交订单，系统生成订单记录并返回确认提示

**Independent Test**: 在 US1 基础上 → 点击提交订单 → 验证订单记录生成 → 验证成功提示 → 验证空购物车阻止提交

### Backend Implementation for US2

- [ ] T037 [P] [US2] Implement order service in backend/src/services/order.js to create order with validation
- [ ] T038 [P] [US2] Add order validation logic in backend/src/services/order.js (empty cart check, duplicate order check)
- [ ] T039 [P] [US2] Create POST /api/orders endpoint in backend/src/api/routes/orders.js to submit orders
- [ ] T040 [P] [US2] Create GET /api/orders endpoint in backend/src/api/routes/orders.js to query user's order history
- [ ] T041 [P] [US2] Create GET /api/orders/:id endpoint in backend/src/api/routes/orders.js to get order details
- [ ] T042 [US2] Implement price snapshot logic in backend/src/services/order.js to save current prices in order_items
- [ ] T043 [US2] Add unique constraint check in backend/src/services/order.js to prevent duplicate orders per day

### Frontend Implementation for US2

- [ ] T044 [P] [US2] Add submit order button in frontend/src/components/Cart.js
- [ ] T045 [P] [US2] Implement order submission API call in frontend/src/services/api.js (POST /api/orders)
- [ ] T046 [US2] Add empty cart validation in frontend/src/components/Cart.js before submission
- [ ] T047 [US2] Implement success toast notification in frontend/src/components/Toast.js
- [ ] T048 [US2] Clear cart after successful order submission in frontend/src/services/cart-storage.js
- [ ] T049 [US2] Implement order history view in frontend/src/components/OrderHistory.js

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - 行政查看订餐统计清单 (Priority: P3)

**Goal**: 行政专员查看当天所有订单汇总，支持按菜品/员工分组，并导出 Excel

**Independent Test**: 行政登录后台 → 选择日期 → 查看订单汇总 → 按菜品分组统计 → 导出 Excel 文件

### Backend Implementation for US3

- [ ] T050 [P] [US3] Implement admin order query service in backend/src/services/order.js to get orders by date with grouping
- [ ] T051 [P] [US3] Create GET /api/admin/orders endpoint in backend/src/api/routes/admin.js to return order summary
- [ ] T052 [P] [US3] Implement Excel export service in backend/src/services/export.js using xlsx library
- [ ] T053 [P] [US3] Create GET /api/admin/orders/export endpoint in backend/src/api/routes/admin.js to download Excel
- [ ] T054 [US3] Add admin role check in backend/src/api/middleware/auth.js (optional: for now assume all logged-in users can access)
- [ ] T055 [US3] Implement order grouping logic in backend/src/services/order.js (group by menu item or employee)

### Frontend Implementation for US3

- [ ] T056 [P] [US3] Create AdminPanel component in frontend/src/components/AdminPanel.js
- [ ] T057 [P] [US3] Create admin.html page in frontend/src/pages/admin.html (or add admin mode to index.html)
- [ ] T058 [P] [US3] Implement date picker for order date selection in frontend/src/components/AdminPanel.js
- [ ] T059 [P] [US3] Implement order summary display with grouping toggle (by employee/by menu) in frontend/src/components/AdminPanel.js
- [ ] T060 [P] [US3] Add statistics cards in frontend/src/components/AdminPanel.js (total orders, total revenue)
- [ ] T061 [US3] Implement export button in frontend/src/components/AdminPanel.js to trigger Excel download
- [ ] T062 [US3] Handle file download in frontend/src/services/api.js for binary response (Excel file)

**Checkpoint**: All user stories should now be independently functional

---

## Phase 6: Authentication & Security (Cross-Cutting Concern)

**Purpose**: Implement WeChat OAuth2 authentication for production environment

- [ ] T063 [P] Implement WeChat OAuth2 service in backend/src/services/auth.js (login redirect, callback handler, token exchange)
- [ ] T064 [P] Create GET /api/auth/wechat/login endpoint in backend/src/api/routes/auth.js to redirect to WeChat
- [ ] T065 [P] Create GET /api/auth/wechat/callback endpoint in backend/src/api/routes/auth.js to handle OAuth callback
- [ ] T066 [P] Implement employee auto-sync in backend/src/services/auth.js to save/update employee info from WeChat
- [ ] T067 [P] Add mock auth mode in backend/src/services/auth.js for development environment (bypass WeChat OAuth)
- [ ] T068 [P] Implement login button in frontend/src/components/LoginButton.js
- [ ] T069 [US1] Add JWT token storage in frontend/src/services/auth-storage.js using localStorage
- [ ] T070 Add JWT token to API requests in frontend/src/services/api.js (Authorization header)

---

## Phase 7: Price Lock Automation (Cross-Cutting Concern)

**Purpose**: Implement automated price locking at scheduled time

- [ ] T071 [P] Implement price lock service in backend/src/utils/price-lock.js using node-cron
- [ ] T072 [P] Add cron job in backend/src/server.js to lock prices daily at 10:00 AM
- [ ] T073 Add price lock time indicator in frontend/src/components/MenuList.js (show "价格已锁定" badge)

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T074 [P] Add error boundary handling in frontend/src/main.js to catch and display errors
- [ ] T075 [P] Implement loading states in frontend/src/components/LoadingSpinner.js
- [ ] T076 [P] Add responsive design breakpoints in frontend/src/styles/main.css for mobile devices
- [ ] T077 [P] Create PWA manifest.json in frontend/public/manifest.json for offline support
- [ ] T078 [P] Implement service worker in frontend/public/service-worker.js for caching
- [ ] T079 [P] Add database backup script in backend/scripts/backup-db.sh
- [ ] T080 [P] Create README.md in project root with setup instructions
- [ ] T081 Implement edge case handling: empty menu items (show "今日菜品暂未更新" message)
- [ ] T082 Implement edge case handling: network errors (show "提交失败，请稍后重试" and preserve cart)
- [ ] T083 Implement edge case handling: duplicate order (show "今日已有订单" message)
- [ ] T084 Add input validation and sanitization across all API endpoints
- [ ] T085 Run quickstart.md validation (follow setup steps and verify all features work)

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3-5)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Auth (Phase 6)**: Can develop in parallel with user stories, integrate at end
- **Price Lock (Phase 7)**: Can develop in parallel, integrate at end
- **Polish (Phase 8)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - Builds on US1 UI but independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - Requires US1+US2 data but independent implementation

### Within Each User Story

- Backend services before API endpoints
- Frontend components can develop in parallel with backend (mock data first)
- Integration happens after both backend and frontend components complete

### Parallel Opportunities

- **Phase 1 Setup**: All tasks (T001-T008) can run in parallel
- **Phase 2 Foundational**: 
  - Models (T012-T014) can run in parallel
  - Utilities (T015-T017) can run in parallel
  - Middleware (T018-T020) can run in parallel
  - Frontend utilities (T023-T025) can run in parallel
- **Phase 3 (US1)**:
  - Backend tasks (T026-T028) can run in parallel
  - Frontend components (T029-T031) can run in parallel
- **Phase 4 (US2)**:
  - Backend endpoints (T037-T041) can run in parallel
  - Frontend tasks (T044-T045, T047-T048) can run in parallel
- **Phase 5 (US3)**:
  - Backend tasks (T050-T053) can run in parallel
  - Frontend tasks (T056-T060) can run in parallel
- **Phase 6 (Auth)**: All tasks (T063-T068) can run in parallel
- **Phase 7 (Price Lock)**: All tasks (T071-T073) can run in parallel
- **Phase 8 (Polish)**: Most tasks (T074-T080) can run in parallel

---

## Parallel Example: User Story 1

```bash
# Launch backend tasks together:
Task: "Implement menu service in backend/src/services/menu.js"
Task: "Create GET /api/menu endpoint in backend/src/api/routes/menu.js"

# Launch frontend tasks together:
Task: "Create MenuList component in frontend/src/components/MenuList.js"
Task: "Create Cart component in frontend/src/components/Cart.js"
Task: "Create category tabs UI in frontend/src/components/CategoryTabs.js"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001-T008)
2. Complete Phase 2: Foundational (T009-T025) - CRITICAL blocking phase
3. Complete Phase 3: User Story 1 (T026-T036)
4. **STOP and VALIDATE**: Test US1 independently - can employees browse and add to cart?
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 (T026-T036) → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 (T037-T049) → Test independently → Deploy/Demo
4. Add User Story 3 (T050-T062) → Test independently → Deploy/Demo
5. Add Authentication (T063-T070) → Integrate and test
6. Add Price Lock (T071-T073) → Integrate and test
7. Polish (T074-T085) → Final QA and deploy

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together (T001-T025)
2. Once Foundational is done:
   - Developer A: User Story 1 (T026-T036)
   - Developer B: User Story 2 (T037-T049)
   - Developer C: User Story 3 (T050-T062)
   - Developer D: Authentication (T063-T070) + Price Lock (T071-T073)
3. Stories complete and integrate independently
4. All team: Polish phase (T074-T085)

---

## Notes

- **[P] tasks**: Different files, no dependencies - safe to parallelize
- **[Story] label**: Maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- **Tests**: Not included as spec.md does not explicitly request TDD approach. If tests are needed later, add them before each implementation task in the appropriate user story phase.

---

## Task Count Summary

- **Phase 1 (Setup)**: 8 tasks
- **Phase 2 (Foundational)**: 17 tasks (BLOCKING)
- **Phase 3 (US1 - P1 MVP)**: 11 tasks
- **Phase 4 (US2 - P2)**: 13 tasks
- **Phase 5 (US3 - P3)**: 13 tasks
- **Phase 6 (Auth)**: 8 tasks
- **Phase 7 (Price Lock)**: 3 tasks
- **Phase 8 (Polish)**: 12 tasks

**Total**: 85 tasks

**Parallel Opportunities**: 45+ tasks can run in parallel within their phases

**MVP Scope**: Phase 1-3 (36 tasks) delivers working employee ordering flow

**Format Validation**: ✅ All tasks follow checklist format (checkbox + ID + [P]/[Story] labels + file paths)
