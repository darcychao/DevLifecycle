# Development Lifecycle Framework

AI 智能体驱动的软件开发生命周期框架。通过标准化流程编排 Scanner、SE、UX、Dev、Test 五大智能体协同工作，内置挑战/纠偏机制、阶段报告体系、需求目录隔离、UX 规范提取与检视、插件扩展系统。

## 架构概览

```
┌──────────────────────────────────────────────────────────────┐
│                    CLAUDE.md (编排器)                         │
├──────────────────────────────────────────────────────────────┤
│  Agents   │ Scanner │ SE │ UX │ Dev │ Test Agents            │
├──────────┼──────────────────────────────────────────────────┤
│  Workflow │  Standard Lifecycle + Challenge + Plugin         │
│           │  + Dev-Story Shortcut + UX Spec + Review         │
├──────────┼──────────────────────────────────────────────────┤
│ Artifacts │  PRD │ SE Design │ UX Spec │ Story │ Test Plan  │
├──────────┼──────────────────────────────────────────────────┤
│  Reports  │  每个阶段产出阶段报告 + 挑战记录文档               │
├──────────┼──────────────────────────────────────────────────┤
│ Standards │  JavaScript │ TypeScript │ Java │ Python         │
│           │  + UX Guidelines (if UI)                         │
├──────────┼──────────────────────────────────────────────────┤
│  Plugins  │  12 hook points at every phase transition        │
└──────────────────────────────────────────────────────────────┘
```

## 生命周期

```
Phase 1    Phase 2    Phase 2.6  Phase 3    Phase 4    Phase 5    Phase 6      Phase 7
┌──────┐  ┌──────┐  ┌───────┐  ┌───────┐  ┌──────┐  ┌──────┐  ┌────────┐  ┌──────────┐
│ PRD  │─▶│  SE  │─▶│  UX   │─▶│ Story │─▶│ Test │─▶│ Dev  │─▶│ Code   │─▶│Validation│
│Input │  │Design│  │ Spec  │  │Design │  │ Plan │  │Coding│  │Review  │  │ (Plugin) │
└──────┘  └──────┘  └───────┘  └───────┘  └──────┘  └──────┘  └────────┘  └──────────┘
  用户      SE智能体   UX智能体   开发智能体  测试智能体  开发智能体  SE+UX智能体  测试智能体
                     (条件执行：
                      UI项目激活)

每个阶段产出: [主制品] + [阶段报告]    每次挑战产出: [挑战记录文档]
```

**双入口：** 全生命周期（PRD 输入）/ Dev-Story 快捷入口（跳过上游直接进入开发编码）

**Phase 2.6 UX Spec Extraction (条件阶段)：** 仅在项目有 UI 且需求涉及界面设计时激活。UX Agent **从已有设计产物中提取** UX 需求级规范（不进行主动设计）。

## 快速开始

### 初始化（每个项目的首次操作，强制要求）

```bash
@CLAUDE.md 初始化框架
```

框架自动执行 8 步初始化协议：检测语言栈 + **UI 框架/UX 依赖** → Scanner 扫描（文件分类、模块识别、依赖分析、**编码风格检测**、**UX 规范检测**）→ 生成文档 → 交叉验证 → 复杂度分级 → 写入锁文件。

生成产物：
- `docs/architecture.md` — 项目架构文档（含 ASCII 架构图、依赖矩阵）
- `docs/module-map.md` — 模块对照表（含关键词→模块反向索引、功能能力标签、文件→模块映射、共享资源目录）
- `docs/coding-standards.md` — 编码规范（含 **§0 项目检测约定**：公共方法、常量定义、文件组织、编码风格、**§0.6 UX规范约定**；冲突时覆盖通用规则）
- `docs/ux-guidelines.md` — **UX 指南**（UI 项目自动生成：UI 技术栈、组件架构、样式约定、布局模式、无障碍基线、设计令牌）
- `docs/project-structure.md` — 项目结构规范
- `docs/.scanner-report.json` — 机器可读的扫描数据（含 UX 堆栈记录）
- `docs/modules/MOD-XXX.md` — 大型项目的分模块详细文档
- `docs/public-method-catalog.md` — 公共方法全量目录（含完整签名、分类、提供模块、消费模块）
- `docs/constant-catalog.md` — 常量全量目录（含名称、类型、值、位置、类别、魔法值报告）
- `docs/terminology-glossary.md` — 领域术语表（含自动推导定义、术语关联、领域聚类）

### 标准开发流程

```bash
@CLAUDE.md 基于PRD开始开发流程
```

### Dev-Story 快捷入口

```bash
@CLAUDE.md 基于Dev Story开始开发
```

---

## 五大智能体

| 智能体 | 职责 | 核心产出 |
|--------|------|---------|
| **Scanner Agent** | 项目扫描、模块识别、依赖分析、公共方法编目、常量编目、领域术语提取、**UX 规范检测** | module-map.md、public-method-catalog.md、constant-catalog.md、terminology-glossary.md、scanner-report.json、coding-standards.md §0（含 §0.6 UX规范约定）、ux-guidelines.md |
| **SE Agent** | 系统架构设计、代码审核（Phase 6 CR-1~CR-8） | SE 设计文档、代码审核报告 |
| **UX Agent** | **UX 需求规范提取**（Phase 2.6，从已有设计产物中提取，不主动设计）、**UX 规范合规审查**（CR-9） | UX 需求级规范 (ux-spec.md)、CR-9 审查报告章节 |
| **Dev Agent** | Story 设计、编码实现、挑战响应 | Dev Story、源代码 |
| **Test Agent** | 测试方案设计、流程验证 | 测试方案、验证报告 |

### UX Agent 核心原则

**UX Agent 从已有设计产物中提取和整理 UX 需求，不进行主动设计。** 所有 UX 规范条目必须可追溯到来源（PRD §5.4、SE Design §4.5、用户提供的设计稿）。无法追溯的需求标记为 GAP 并通过 CAT-1 挑战请求用户补充，绝不自行发明。

**两大工作模式：**

| 模式 | 阶段 | 说明 |
|------|------|------|
| **UX Spec Extraction** | Phase 2.6 | 从 PRD UX 约束、SE Design UI 架构、设计稿、ux-guidelines.md 中提取结构化 UX 需求规范 |
| **UX Spec Compliance Review** | Phase 6 (CR-9) | 6 项子检查：视觉需求合规、交互需求合规、响应式需求合规、无障碍需求合规、设计系统合规、UX 状态完整性 |

---

## 阶段详情

| 阶段 | 智能体 | 输入 | 制品 | 审查人 |
|------|--------|------|------|--------|
| 1. PRD | 用户 | 业务需求 | `prd.md`（含 §5.4 UX 约束） | — |
| 2. SE Design | SE Agent | PRD、架构文档 | `se-design.md`（含 §4.5 UI 架构） | Dev Agent |
| **2.6 UX Spec** | **UX Agent** | **PRD §5.4、SE Design §4.5、设计稿、UX 指南** | **`ux-spec.md`**（提取，不设计） | **Dev Agent** |
| 3. Story Design | Dev Agent | SE Design、PRD、UX Spec | `dev-story.md`（含 §6.1 UX 合规检查） | SE Agent + UX Agent |
| 4. Test Plan | Test Agent | PRD、SE Design、Dev Story | `test-plan.md` | Dev Agent + 用户 |
| 5. Dev Coding | Dev Agent | Dev Story、Test Plan、UX Spec | 源代码 | SE Agent (Phase 6) |
| 5.5 Catalog Self-Check | Dev Agent | 代码差异、目录 | 自检（嵌入 Phase 5 Report） | — |
| 6. Code Review | SE + UX Agent | 代码、Dev Story、Test Plan、UX Spec | 审核报告（CR-1~CR-9） | — |
| 6.5 Catalog Verification | SE Agent | 代码差异、目录 | 一致性核查 | — |
| 7. Validation | Test Agent | 代码、Test Plan、审核报告 | 验证报告 | — |

**关键关卡：** Phase 0 锁文件 → Phase 2.6 UX 规范提取（UI 项目）→ Phase 5.5 目录自检 → Phase 6（CR-1~CR-9）→ Phase 6.5 目录一致性 → Phase 7 验证

---

## 代码审核（Phase 6 硬关卡）

SE Agent 执行 CR-1~CR-8，UX Agent 执行 CR-9（UI 项目）：

| 检查项 | 审查者 | 内容 |
|--------|--------|------|
| CR-1 需求完整性 | SE Agent | 所有需求均已实现，无 TODO/stub |
| CR-2 Story 对齐 | SE Agent | 文件路径、函数签名与 Story 一致 |
| CR-3 规范合规 | SE Agent | 命名、格式、禁止项、§0.6 UX规范约定 |
| CR-4 架构完整性 | SE Agent | 无循环依赖、模块规则合规 |
| CR-5 逻辑正确性 | SE Agent | 错误处理、边界条件、竞态 |
| CR-6 测试覆盖 | SE Agent | AAA 模式、正常/异常/边界路径 |
| CR-7 安全性 | SE Agent | 无硬编码密钥、注入防护 |
| CR-8 无遗漏 | SE Agent | 无注释代码、无空 catch、无死代码 |
| **CR-9 UX 规范合规** | **UX Agent** | **视觉需求、交互需求、响应式、无障碍、设计系统、UX 状态完整性** |

**任一 FAIL → 自动挑战 → 生成挑战记录 → 修复 → 重审 → 全部 PASS 才能进入 Phase 7。**

CR-9 仅审查 UX 规范中 **明确规定的条目**。规范未涵盖的 UI 方面记录为"超出审查范围"，不作为失败项。

---

## UX 约束与规范体系

### 强制 UX 约束（`framework/workflows/ux-constraint.rule.md`）

当项目有 UI 时，PRD 和 Dev Story 涉及界面设计时必须满足：

**PRD 阶段 (§5.4)：**

| 约束 | 内容 | 性质 |
|------|------|------|
| UX-01 | 交互流程（页面导航图、用户操作路径） | 阻止性 |
| UX-02 | 视觉参考（设计稿链接、线框图、样式参考） | 阻止性 |
| UX-03 | 响应式设计要求（设备类型、断点、布局差异） | 阻止性 |
| UX-04 | 表单交互规范（验证规则、错误提示、提交行为） | 阻止性（如有表单） |
| UX-05 | 状态反馈说明（加载中、空状态、错误状态） | 阻止性 |
| UX-06 | 可访问性要求（键盘、屏幕阅读器、色彩对比度） | 警告 |
| UX-07 | 动效与过渡要求 | 警告 |

**Phase 2.6 UX 规范提取：** UX Agent 从上述约束 + SE Design UI 架构 + 设计稿中提取结构化 UX 需求规范，标记空白并请求用户补充。

**Phase 6 CR-9 审查：** 仅基于已提取的 UX 规范条目进行合规检查，不在规范范围内的 UI 决策不构成失败。

---

## 核心机制

### 挑战机制（最高优先级 — 核心迭代引擎）

三大主动挑战类别：

| 类别 | 名称 | 触发条件 | 挑战方 |
|------|------|---------|--------|
| **CAT-1** | 需求遗漏 | 下游发现上游制品缺少功能需求或设计元素 | 任何智能体 |
| **CAT-2** | 编码规范违反 | 代码或设计违反项目编码规范 / 目录冲突 | 任何智能体 |
| **CAT-3** | **UX 规范违反** | **代码偏离已提取的 UX 规范条目（仅限规范明确规定的项目）** | **UX Agent** |

**CAT-3 约束：** 只能基于 UX 规范中明确规定的条目发起挑战。基于个人美学偏好的挑战将**被驳回**。规范未涵盖的 UI 方面不构成 CAT-3。

详见: `framework/workflows/challenge-mechanism.rule.md`

### 插件扩展机制

12 个 Hook 点分布在每个阶段转换处，允许项目注入自定义行为。

详见: `framework/workflows/plugin-extension.rule.md`

---

## 编码规范（二级优先级体系 + UX）

```
Tier 1 (最高优先级)    Tier 2 (通用兜底)
┌──────────────────┐   ┌──────────────────────────┐
│ docs/            │   │ framework/standards/      │
│ coding-standards │   │ coding-standards.<lang>.md │
│ .md §0           │──▶│ (15 章通用语言规范)        │
│ (项目检测约定)     │   │                          │
├──────────────────┤   └──────────────────────────┘
│ §0.2 公共方法约定  │
│ §0.3 常量定义约定  │
│ §0.4 文件组织约定  │
│ §0.5 编码风格约定  │
│ §0.6 UX规范约定   │ ← UI 项目自动检测生成
└──────────────────┘
  优先遵循，冲突时覆盖 Tier 2
```

### §0.6 UX规范约定（UI 项目自动生成）

| 检测类别 | 检测内容 |
|----------|---------|
| UI Framework & Libraries | React/Vue/Angular 版本、组件库、样式方案、状态管理、路由 |
| 组件命名约定 | 组件文件命名、导出方式、Props 模式 |
| 样式约定 | Tailwind class 顺序 / CSS Module 命名 / styled-component 模式、响应式断点 |
| 布局约定 | Flexbox/Grid 使用、容器模式、间距尺度 |
| 可访问性约定 | ARIA 使用率、语义化 HTML、alt 文本、键盘处理、i18n 模式 |
| 设计系统集成 | 颜色/字体/间距的设计令牌、组件变体、主题模式 |

---

## 目录驱动的开发流程

Phase 0 初始化生成的四份目录是下游智能体的**权威参考数据源**：

| 目录 | 用途 | 强制规则 |
|------|------|---------|
| `public-method-catalog.md` | 避免方法重复 | MUST-01~MUST-04 |
| `constant-catalog.md` | 避免常量重复和魔法值 | MUST-05~MUST-08 |
| `terminology-glossary.md` | 确保命名和语义统一 | MUST-09~MUST-12 |
| `ux-guidelines.md` | UX 规范基准（UI 项目） | §0.6 强制约定 |

详见: `framework/workflows/coding-standards-hierarchy.rule.md`

---

## 目录结构

```
project/
├── CLAUDE.md                              # 主编排器
├── README.md                              # 本文件
├── .claude/settings.json                  # Claude Code 配置
├── framework/
│   ├── agents/                            # 智能体技能定义
│   │   ├── project-scanner.skill.md        # Scanner Agent
│   │   ├── se-agent.skill.md               # SE Agent
│   │   ├── ux-agent.skill.md               # UX Agent (NEW)
│   │   ├── dev-agent.skill.md              # Dev Agent
│   │   └── test-agent.skill.md             # Test Agent
│   ├── workflows/                         # 工作流规则 (8 个文件)
│   │   ├── challenge-mechanism.rule.md     # 挑战/纠偏机制 (最高优先级)
│   │   ├── plugin-extension.rule.md       # 插件扩展机制
│   │   ├── ux-constraint.rule.md           # UX 约束强制机制 (NEW)
│   │   ├── standard-lifecycle.rule.md     # 标准生命周期 (含 Phase 5.5/6.5/UX)
│   │   ├── execution-protocol.rule.md     # 执行协议 (NEW)
│   │   ├── initialization.rule.md         # Phase 0 初始化协议 (NEW)
│   │   ├── requirement-model.rule.md      # 需求目录模型 (NEW)
│   │   └── coding-standards-hierarchy.rule.md  # 编码规范层级 (NEW)
│   ├── artifacts/                         # 制品模板 (9 个)
│   │   ├── prd.template.md
│   │   ├── se-design.template.md
│   │   ├── ux-spec.template.md            # UX 需求规范模板 (NEW)
│   │   ├── dev-story.template.md
│   │   ├── test-plan.template.md
│   │   ├── architecture-doc.template.md
│   │   ├── module-detail.template.md
│   │   ├── public-method-catalog.template.md
│   │   ├── constant-catalog.template.md
│   │   └── terminology-glossary.template.md
│   └── standards/                         # 规范文档
│       ├── coding-standards.template.md
│       ├── coding-standards.javascript.md
│       ├── coding-standards.typescript.md
│       ├── coding-standards.java.md
│       ├── coding-standards.python.md
│       ├── project-structure.template.md
│       └── validation-standards.template.md
├── plugins/                               # 插件扩展
└── docs/                                  # 生成文档
    ├── .framework-init.lock               # 初始化锁文件（JSON，含 ui_stack）
    ├── architecture.md                    # 项目架构
    ├── module-map.md                      # 模块对照表（含关键词索引）
    ├── coding-standards.md                # 编码规范（§0 含 §0.6 UX）
    ├── ux-guidelines.md                   # UX 指南（UI 项目自动生成）
    ├── project-structure.md               # 项目结构
    ├── public-method-catalog.md           # 公共方法目录
    ├── constant-catalog.md                # 常量目录
    ├── terminology-glossary.md            # 领域术语表
    ├── .scanner-report.json               # 扫描数据（机器可读）
    ├── challenges/
    │   └── index.md                       # 全局挑战索引
    ├── modules/                           # 分模块文档（大型项目）
    └── requirements/                      # 需求隔离目录
        ├── index.md                       # 需求注册表
        └── REQ-YYYY-NNN-{slug}/
            ├── prd.md
            ├── se-design.md
            ├── ux-spec.md                 # UX 需求规范（UI 需求）
            ├── dev-story.md
            ├── test-plan.md
            ├── validation-report.md
            ├── reports/                   # 阶段报告（含 phase-2.6-ux-spec-report.md）
            └── challenges/                # 挑战记录
```

---

## 工作流规则文件速查

| 优先级 | 文件 | 用途 |
|--------|------|------|
| 0 | `challenge-mechanism.rule.md` | 挑战/纠偏机制（CAT-1/CAT-2/CAT-3） |
| 1 | `plugin-extension.rule.md` | 12 个 Hook 点插件注入 |
| 2 | `ux-constraint.rule.md` | UX 约束强制（PRD/Story UI 设计关卡） |
| 3 | `standard-lifecycle.rule.md` | 7 阶段流程 + 快捷路径 + Phase 5.5/6.5 目录关卡 |
| 4 | `execution-protocol.rule.md` | 9 步执行序列 |
| 5 | `initialization.rule.md` | Phase 0 协议（含 UX 堆栈检测 Step 0.1.1） |
| 6 | `requirement-model.rule.md` | 需求目录模型、REQ-ID 分配 |
| 7 | `coding-standards-hierarchy.rule.md` | 二级编码规范 + §0.6 UX规范约定 |

---

## 设计原则

1. **模板驱动** — 所有智能体产出遵循统一模板，标准违背可被机械检测
2. **挑战驱动迭代** — 三大挑战类别（CAT-1 需求遗漏、CAT-2 规范违反、CAT-3 UX 规范违反）构成自校正迭代引擎
3. **提取而非设计** — UX Agent 从已有设计产物中提取 UX 需求规范，不主动设计；CAT-3 只能基于明确规范条目
4. **扫描驱动** — 项目结构、共享资源、模块依赖、UX 规范由 Scanner Agent 自动检测，编码规范由扫描结果驱动
5. **需求隔离** — 每个需求拥有独立目录，制品/报告/挑战自包含，支持并行开发
6. **阶段报告** — 每个阶段强制产出报告，形成完整的开发过程审计链
7. **挑战留痕** — 每次挑战记录为独立文档，永久留存，可追溯
8. **强制初始化** — 所有生命周期操作前检查锁文件，确保项目信息（含 UX 堆栈）完整准确
9. **目录完整性** — 公共方法、常量、术语、UX 指南四份目录构成权威参考数据源，智能体编码前必须查阅
