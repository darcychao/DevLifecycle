# Development Lifecycle Framework

AI 智能体驱动的软件开发生命周期框架。通过标准化的 7 阶段流程编排 SE、Dev、Test 三大智能体协同工作，内置挑战/纠偏机制、阶段报告体系、需求目录隔离和插件扩展系统。

## 架构概览

```
┌──────────────────────────────────────────────────────────────┐
│                    CLAUDE.md (编排器)                         │
├──────────────────────────────────────────────────────────────┤
│  Agents   │ Scanner │ SE Agent │ Dev Agent │ Test Agent        │
├──────────┼──────────────────────────────────────────────────┤
│  Workflow │  Standard Lifecycle + Challenge + Plugin         │
│           │  + Dev-Story Shortcut                            │
├──────────┼──────────────────────────────────────────────────┤
│ Artifacts │  PRD │ SE Design │ Story │ Test Plan             │
├──────────┼──────────────────────────────────────────────────┤
│  Reports  │  每个阶段产出阶段报告 + 挑战记录文档               │
├──────────┼──────────────────────────────────────────────────┤
│ Standards │  JavaScript │ TypeScript │ Java │ Python         │
├──────────┼──────────────────────────────────────────────────┤
│  Plugins  │  12 hook points at every phase transition        │
└──────────────────────────────────────────────────────────────┘
```

## 生命周期

```
Phase 1    Phase 2    Phase 3    Phase 4    Phase 5    Phase 6      Phase 7
┌──────┐  ┌──────┐  ┌───────┐  ┌──────┐  ┌──────┐  ┌────────┐  ┌──────────┐
│ PRD  │─▶│  SE  │─▶│ Story │─▶│ Test │─▶│ Dev  │─▶│ Code   │─▶│Validation│
│Input │  │Design│  │Design │  │ Plan │  │Coding│  │Review  │  │ (Plugin) │
└──────┘  └──────┘  └───────┘  └──────┘  └──────┘  └────────┘  └──────────┘
  用户      SE智能体   开发智能体   测试智能体  开发智能体   SE智能体     测试智能体

每个阶段产出: [主制品] + [阶段报告]    每次挑战产出: [挑战记录文档]
```

**双入口：** 全生命周期（PRD 输入）/ Dev-Story 快捷入口（跳过上游直接进入开发编码）

## 快速开始

### 初始化（每个项目的首次操作，强制要求）

```bash
@CLAUDE.md 初始化框架
```

框架自动执行 8 步初始化协议：检测语言栈 → Scanner 扫描（文件分类、模块识别、依赖分析、**编码风格检测**）→ 生成文档 → 交叉验证 → 复杂度分级 → 写入锁文件。

生成产物：
- `docs/architecture.md` — 项目架构文档（含 ASCII 架构图、依赖矩阵）
- `docs/module-map.md` — 模块对照表（文件→模块映射、公共 API 清单、共享资源目录）
- `docs/coding-standards.md` — 编码规范（**含 §0 项目检测约定**：公共方法、文件组织、编码风格，自动检测覆盖率，冲突时覆盖通用规则）
- `docs/project-structure.md` — 项目结构规范
- `docs/.scanner-report.json` — 机器可读的扫描数据（含检测约定，供下游智能体消费）

### 标准开发流程

```bash
@CLAUDE.md 基于PRD开始开发流程
```

### Dev-Story 快捷入口

```bash
@CLAUDE.md 基于Dev Story开始开发
```

## 需求目录模型

每个需求/特性拥有独立的隔离目录，所有制品、报告、挑战记录自包含：

```
docs/requirements/
├── index.md                          # 需求注册表（全局）
└── REQ-2026-001-user-auth/           # 需求隔离目录
    ├── prd.md                        # Phase 1 制品
    ├── se-design.md                  # Phase 2 制品
    ├── dev-story.md                  # Phase 3 制品
    ├── test-plan.md                  # Phase 4 制品
    ├── validation-report.md          # Phase 7 制品
    ├── reports/                      # 阶段完成报告
    │   ├── phase-1-prd-report.md
    │   ├── phase-2-se-design-report.md
    │   ├── phase-3-story-design-report.md
    │   ├── phase-4-test-plan-report.md
    │   ├── phase-5-dev-coding-report.md
    │   ├── phase-6-code-review-report.md
    │   └── phase-7-validation-report.md
    └── challenges/                   # 挑战记录
        └── CH-YYYY-NNN.md
```

**共享文档**（`docs/` 根目录）：
- `architecture.md`, `module-map.md`, `coding-standards.md`, `project-structure.md` — Phase 0 初始化产出，所有需求共享
- `challenges/index.md` — 全局挑战索引（跨需求汇总）

**多需求并行：** 多个 REQ 目录可同时存在，各自独立推进，互不干扰。

## 阶段报告体系

每个阶段完成后 **必须** 生成阶段报告，记录执行详情：

| 报告 | 内容 |
|------|------|
| 输入制品清单 | 本阶段依赖的文档及其状态 |
| 执行摘要 | 执行步骤数、关键决策、偏离说明 |
| 输出制品清单 | 产出文件及规模 |
| 质量门检查 | 模板合规、章节完整、审查人确认 |
| 插件执行记录 | pre/post hook 执行状态 |
| 审查人签字 | ACCEPT / CHALLENGE |

阶段报告是阶段完成的 **强制凭证** — 无报告不得进入下一阶段。

## 挑战文档体系

每次挑战 **必须** 记录为独立文档，包含：

| 字段 | 说明 |
|------|------|
| Basis | 具体规范条款引用 |
| Evidence | 违规代码路径:行号 |
| Impact | 影响范围和风险 |
| Suggested Fix | 具体修复方案 |
| Resolution | 修复内容、验证方式、重审结果 |

**挑战生命周期：** OPEN → RESOLVED → 验证关闭 / 重新打开 → ESCALATED（2轮僵持后升级给用户）/ REJECTED（无效挑战）

**全局索引**（`docs/challenges/index.md`）记录所有需求的所有挑战，支持跨需求追溯。

## 四大智能体

| 智能体 | 职责 | 核心产出 |
|--------|------|---------|
| **Scanner Agent** | 项目结构扫描、模块边界识别、共享资源检测、依赖分析、**编码风格检测** | module-map.md、scanner-report.json、coding-standards.md §0（检测约定） |
| **SE Agent** | 系统架构设计、代码审核（Phase 6 硬关卡） | SE 设计文档、代码审核报告 |
| **Dev Agent** | Story 设计、编码实现、挑战响应 | Dev Story、源代码 |
| **Test Agent** | 测试方案设计、流程验证 | 测试方案、验证报告 |

## 核心机制

### 挑战机制（最高优先级 — 核心迭代引擎）

挑战机制是框架的 **自校正迭代引擎**。智能体在正常执行过程中 **主动** 发现并报告问题，而非等待审查阶段才暴露。框架将挑战视为积极行为 — 不是失败信号，而是质量保证的手段。

**两大主动挑战类别：**

| 类别 | 名称 | 触发条件 | 示例 |
|------|------|---------|------|
| **CAT-1** | 需求遗漏 | 下游智能体发现上游制品缺失功能需求或设计元素 | PRD 有 FR-005 但 SE Design 无对应模块 |
| **CAT-2** | 编码规范违反 | 任何智能体发现代码或设计违反项目编码规范 | 新文件放入错误目录、命名风格与 §0 检测约定不一致 |

**迭代流程：** 智能体执行 → 主动检测 CAT-1/CAT-2 → 发现即发起挑战 → 被挑战方立即修复 → 重新执行 → 通过后继续推进。

**核心规则：**
- 发现需求遗漏或规范违反时 **立即发起** 挑战，不等下游阶段暴露
- 挑战必须引用具体规范条款（PRD FR-XXX / coding-standards.md §X.Y）+ 文件:行号
- 被挑战方 **必须停止当前工作** 优先处理
- 2 轮僵持后升级给用户决策
- Code Review（Phase 6）的 8 项检查失败自动触发系统生成型挑战
- 每次挑战生成独立文档，永久留存

详见: `framework/workflows/challenge-mechanism.rule.md`

### 代码审核（Phase 6 硬关卡）

SE Agent 对 Dev Agent 产出的代码执行 8 项强制检查：

| 检查项 | 内容 |
|--------|------|
| CR-1 需求完整性 | 所有需求均已实现，无 TODO/stub |
| CR-2 Story 对齐 | 文件路径、函数签名与 Story 一致 |
| CR-3 规范合规 | 命名、格式、禁止项检查 |
| CR-4 架构完整性 | 无循环依赖、模块规则合规 |
| CR-5 逻辑正确性 | 错误处理、边界条件、竞态 |
| CR-6 测试覆盖 | AAA 模式、正常/异常/边界路径 |
| CR-7 安全性 | 无硬编码密钥、注入防护 |
| CR-8 无遗漏 | 无注释代码、无空 catch、无死代码 |

**任一 FAIL → 自动挑战 → 生成挑战记录 → 修复 → 重审 → 全部 PASS 才能进入 Phase 7。**

### 插件扩展机制

12 个 Hook 点分布在每个阶段转换处，允许项目注入自定义行为（校验规则、自动化脚本、补充文档等）。

详见: `plugins/README.md`

## 编码规范（预置 4 种语言）

| 语言 | 文件 | 章节 |
|------|------|------|
| JavaScript (ES2022+) | `framework/standards/coding-standards.javascript.md` | 15 章 + 命名速查表 + 禁止项清单 |
| TypeScript (5.x+) | `framework/standards/coding-standards.typescript.md` | **§0 项目检测约定** + 15 章 + 模块组织规则（§7.4-7.6） |
| Java (17 LTS+) | `framework/standards/coding-standards.java.md` | 15 章 + 命名速查表 + 禁止项清单 |
| Python (3.11+) | `framework/standards/coding-standards.python.md` | 15 章 + 命名速查表 + 禁止项清单 |

**§0 项目检测约定** 由 Scanner Agent 在初始化时自动生成，从实际代码中检测公共方法约定（命名、异步、错误模式）、文件组织约定（目录/文件命名、桶导出、Feature 结构）、编码风格约定（缩进、引号、分号、函数风格、导入排序、null处理、行宽）。覆盖率 >70% 认定为项目主导约定；与通用标准冲突时覆盖（⚠ overridden）；不一致项标记为需整改（⬢ remediation）。

**扫描驱动的模块组织规则**（§7.4-7.6）定义了新增代码必须遵守的结构约束：组织模式遵循、共享组件/方法放置规则、模块依赖规则、路径别名使用、公共 API 管理、新增文件检查清单。

其他语言在初始化时从通用模板 `framework/standards/coding-standards.template.md` 自动生成。

## 目录结构

```
H5LifecycleTemplate/
├── CLAUDE.md                              # 主编排器（框架入口）
├── README.md                              # 本文件
├── .claude/settings.json                  # Claude Code 配置
├── framework/
│   ├── agents/                            # 智能体技能定义
│   │   ├── project-scanner.skill.md        # 项目结构扫描
│   │   ├── se-agent.skill.md
│   │   ├── dev-agent.skill.md
│   │   └── test-agent.skill.md
│   ├── workflows/                         # 工作流规则
│   │   ├── standard-lifecycle.rule.md     # 标准生命周期
│   │   ├── challenge-mechanism.rule.md    # 挑战/纠偏机制
│   │   └── plugin-extension.rule.md       # 插件扩展机制
│   ├── artifacts/                         # 制品模板
│   │   ├── prd.template.md
│   │   ├── se-design.template.md
│   │   ├── dev-story.template.md
│   │   ├── test-plan.template.md
│   │   └── architecture-doc.template.md
│   └── standards/                         # 规范文档
│       ├── coding-standards.template.md
│       ├── coding-standards.javascript.md
│       ├── coding-standards.typescript.md
│       ├── coding-standards.java.md
│       ├── coding-standards.python.md
│       ├── project-structure.template.md
│       └── validation-standards.template.md
├── plugins/                               # 插件扩展
│   ├── README.md
│   └── example-plugin/
└── docs/                                  # 生成文档
    ├── .framework-init.lock               # 初始化锁文件（JSON）
    ├── architecture.md                    # 共享 — 项目架构（Phase 0）
    ├── module-map.md                      # 共享 — 模块对照表（Phase 0）
    ├── coding-standards.md                # 共享 — 编码规范（Phase 0）
    ├── project-structure.md               # 共享 — 项目结构（Phase 0）
    ├── .scanner-report.json               # 共享 — 扫描数据（Phase 0，机器可读）
    ├── challenges/
    │   └── index.md                       # 全局挑战索引（跨需求）
    ├── modules/                           # 大型项目分模块文档（Phase 0）
    └── requirements/                      # 需求目录根
        ├── index.md                       # 需求注册表
        └── REQ-YYYY-NNN-{slug}/           # 需求隔离目录
            ├── prd.md, se-design.md, dev-story.md, test-plan.md
            ├── reports/                   # 阶段报告
            └── challenges/                # 挑战记录
```

## 设计原则

1. **模板驱动** — 所有智能体产出遵循统一模板，标准违背可被机械检测
2. **挑战驱动迭代** — 挑战是核心迭代引擎，智能体主动发现需求遗漏(CAT-1)和规范违反(CAT-2)时立即发起，不等下游暴露
3. **扫描驱动** — 项目结构、共享资源、模块依赖由 Scanner Agent 自动检测，编码规范中的模块组织规则由扫描结果驱动执行
4. **需求隔离** — 每个需求拥有独立目录，制品/报告/挑战自包含，支持并行开发
5. **阶段报告** — 每个阶段强制产出报告，形成完整的开发过程审计链
6. **挑战留痕** — 每次挑战记录为独立文档，永久留存，可追溯
7. **Hook 注入** — 12 个插件 Hook 点覆盖每个阶段转换，不修改核心流程即可扩展
8. **强制初始化** — 所有生命周期操作前检查锁文件，确保项目信息完整准确
