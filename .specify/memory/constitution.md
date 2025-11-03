<!--
  SYNC IMPACT REPORT
  ==================
  Version Change: [initial] → 1.0.0
  
  Modified Principles:
  - ALL PRINCIPLES: Initial constitution creation based on user writing rules
  
  Added Sections:
  - Core Principles (5 principles)
  - Development Standards
  - Governance
  
  Removed Sections: None (initial version)
  
  Templates Status:
  ✅ spec-template.md: Reviewed - compliant with writing style (短句、白话释义、验收标准)
  ✅ plan-template.md: Reviewed - compliant with constitution check section
  ✅ tasks-template.md: Reviewed - compliant with user story structure
  ✅ checklist-template.md: Reviewed - compliant with clear action items
  
  Follow-up TODOs: None
-->

# Order Food 产品规范 Constitution

## Core Principles

### I. 受众优先 (Audience-First Writing)

文档受众是"新入职的前端/后端/QA/设计同学"。
所有规范文档必须：
- 使用短句表达（每段 ≤5 行）
- 专业词首次出现时给出白话释义（例：MVP = 最小可行产品）
- 避免行业黑话和假设性知识

**理由**：降低新人理解门槛，加快团队协作效率。

**验收**：新同学能在15分钟内理解核心需求，无需额外解释。

### II. 需求三要素 (Requirement Triad)

每个需求点必须包含：
1. **是什么**：功能描述，用户可见的行为
2. **为什么**：业务价值或用户痛点
3. **怎么验收**：可测试的通过标准（Given-When-Then 或具体指标）

**理由**：避免需求模糊，减少返工和争议。

**验收**：QA 可直接根据需求写测试用例，无需补充提问。

### III. 禁止空话 (Zero-Fluff Policy)

严禁使用：
- 无证据的形容词："全面提升"、"显著增强"、"大幅优化"
- 模糊的目标："改善用户体验"、"提高系统性能"
- 未量化的承诺："尽可能快"、"基本满足"

**理由**：空话无法验收，浪费开发和测试时间。

**验收**：所有描述可被测量或观察，否则删除或改写为具体指标。

### IV. 流程最短化 (Minimal Flow Principle)

所有产品流程必须：
- 用户操作步骤最少化（能 1 步完成不用 2 步）
- 表单字段最必要化（非必需信息后置或删除）
- 等待时间最短化（优先异步处理、预加载）

**理由**：每增加一步操作，流失率提升 20-40%（行业数据）。

**验收**：关键路径（如下单）≤3 步，每步耗时 <5 秒。

### V. 隐私与合规 (Privacy & Compliance - NON-NEGOTIABLE)

产品功能必须：
- 符合当地法律法规（数据保护、消费者权益、食品安全）
- 用户隐私优先（最小权限请求、数据加密存储）
- 安全优先于便利性（支付流程、敏感信息处理）

**理由**：法律风险和用户信任损失无法修复。

**验收**：
- 每个功能通过隐私影响评估（PIA）
- 敏感数据（地址、支付）加密传输和存储
- 用户可随时删除个人数据

## Development Standards

### 上下文摘要 (Context Summary)

所有 spec.md 文件最前方必须包含《上下文摘要》：
- **需求来源**：用户反馈/业务目标/技术升级
- **影响范围**：前端/后端/设计/数据库
- **依赖关系**：依赖或被依赖的功能模块
- **风险提示**：已知技术难点或业务限制

### 澄清清单 (Clarification Checklist)

当需求不明确时，必须输出"澄清清单"而非假设：
- 列出所有不确定点（技术选型、数据来源、边界条件）
- 提供可选方案（A/B/C 方案对比）
- 标注优先级（P0 必须澄清 / P1 可后续确认）

**验收**：清单交付后，产品经理可直接回答每个问题，无需二次解释。

### 表格规范 (Table Standards)

所有表格使用 Markdown 格式：
```
| 字段名 | 类型 | 必填 | 说明 | 示例 |
|--------|------|------|------|------|
| userId | string | 是 | 用户唯一标识 | "u_123456" |
```

**理由**：统一格式，方便版本对比和自动化解析。

## Governance

### 优先级定义

宪法原则的优先级（发生冲突时）：
1. **V. 隐私与合规** - 绝对优先，不可协商
2. **IV. 流程最短化** - 次优先，但不能违反合规
3. **I-III** - 可根据项目阶段调整（早期侧重 II 需求三要素，成熟期侧重 I 受众优先）

### 修订流程

宪法修订需要：
1. 提案人说明修改原因（具体案例 + 改进证据）
2. 团队评审（至少 1 名前端/后端/QA/产品代表）
3. 试运行 2 个功能迭代后正式纳入
4. 更新 `LAST_AMENDED_DATE` 和版本号

### 版本语义

- **MAJOR (X.0.0)**：删除或重新定义现有原则（向后不兼容）
- **MINOR (1.X.0)**：新增原则或大幅扩展章节
- **PATCH (1.0.X)**：澄清措辞、修正错误、补充示例

### 合规检查

每个功能的 plan.md 必须包含 **Constitution Check** 章节：
- 逐条检查适用的原则（特别是 V. 隐私与合规）
- 对违反项提供豁免理由（技术限制/业务权衡）
- QA 在测试前必须确认 Constitution Check 通过

**版本**: 1.0.0 | **Ratified**: 2025-11-03 | **Last Amended**: 2025-11-03
