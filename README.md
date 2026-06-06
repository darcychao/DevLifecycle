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

框架自动执行 8 步初始化协议：检测语言栈 → 扫描文件 → 识别模块边界 → 分析依赖 → 生成文档 → 交叉验证 → 复杂度分级 → 写入锁文件。

生成产物：
- `docs/architecture.md` — 项目架构文档（含 ASCII 架构图、依赖矩阵）
- `docs/module-map.md` — 模块对照表（文件→模块映射、公共 API 清单、共享资源目录）
- `docs/coding-standards.md` — 编码规范（按语言自动匹配，含扫描驱动的模块组织规则）
- `docs/project-structure.md` — 项目结构规范
- `docs/.scanner-report.json` — 机器可读的扫描数据（供下游智能体消费）

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
| **Scanner Agent** | 项目结构扫描、模块边界识别、共享资源检测、依赖分析 | module-map.md、scanner-report.json、架构数据 |
| **SE Agent** | 系统架构设计、代码审核（Phase 6 硬关卡） | SE 设计文档、代码审核报告 |
| **Dev Agent** | Story 设计、编码实现、挑战响应 | Dev Story、源代码 |
| **Test Agent** | 测试方案设计、流程验证 | 测试方案、验证报告 |

## 核心机制

### 挑战机制（最高优先级）

下游智能体发现上游产出违反规范/标准时，可发起正式挑战。挑战必须引用具体规范条款，被挑战方必须立即停止当前工作优先处理。代码审核阶段（Phase 6）的 8 项检查失败会自动触发系统生成型挑战。每次挑战生成独立文档，永久留存。

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
| TypeScript (5.x+) | `framework/standards/coding-standards.typescript.md` | 15 章 + 扫描驱动的模块组织规则（§7.4-7.6） |
| Java (17 LTS+) | `framework/standards/coding-standards.java.md` | 15 章 + 命名速查表 + 禁止项清单 |
| Python (3.11+) | `framework/standards/coding-standards.python.md` | 15 章 + 命名速查表 + 禁止项清单 |

**扫描驱动的模块组织规则**（§7.4-7.6）定义了新增代码必须遵守的结构约束：组织模式遵循、共享组件/方法放置规则、模块依赖规则、路径别名使用、公共 API 管理、新增文件检查清单。这些规则由 Scanner Agent 的扫描结果驱动，确保代码结构与项目实际架构一致。

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
2. **挑战优先** — 纠偏机制优先级最高，防止错误级联
3. **扫描驱动** — 项目结构、共享资源、模块依赖由 Scanner Agent 自动检测，编码规范中的模块组织规则由扫描结果驱动执行
4. **需求隔离** — 每个需求拥有独立目录，制品/报告/挑战自包含，支持并行开发
5. **阶段报告** — 每个阶段强制产出报告，形成完整的开发过程审计链
6. **挑战留痕** — 每次挑战记录为独立文档，永久留存，可追溯
7. **Hook 注入** — 12 个插件 Hook 点覆盖每个阶段转换，不修改核心流程即可扩展
8. **强制初始化** — 所有生命周期操作前检查锁文件，确保项目信息完整准确
