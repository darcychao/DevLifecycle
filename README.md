# H5 Lifecycle Framework

AI 智能体驱动的软件开发生命周期框架。通过标准化的 7 阶段流程编排 SE、Dev、Test 三大智能体协同工作，内置挑战/纠偏机制和插件扩展系统。

## 架构概览

```
┌──────────────────────────────────────────────────────────────┐
│                    CLAUDE.md (编排器)                         │
├──────────────────────────────────────────────────────────────┤
│  Agents   │  SE Agent  │  Dev Agent  │  Test Agent           │
├──────────┼──────────────────────────────────────────────────┤
│  Workflow │  Standard Lifecycle + Challenge + Plugin         │
│           │  + Dev-Story Shortcut                            │
├──────────┼──────────────────────────────────────────────────┤
│ Artifacts │  PRD │ SE Design │ Story │ Test Plan             │
├──────────┼──────────────────────────────────────────────────┤
│ Standards │  JavaScript │ TypeScript │ Java │ Python         │
│           │  + Structure + Validation                        │
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
- `docs/module-map.md` — 模块对照表（文件→模块映射、公共 API 清单）
- `docs/coding-standards.md` — 编码规范（按语言自动匹配）
- `docs/project-structure.md` — 项目结构规范

### 标准开发流程

```bash
@CLAUDE.md 基于PRD开始开发流程
```

### Dev-Story 快捷入口

```bash
@CLAUDE.md 基于Dev Story开始开发
```

## 三大智能体

| 智能体 | 职责 | 核心产出 |
|--------|------|---------|
| **SE Agent** | 系统架构设计、代码审核（Phase 6 硬关卡） | SE 设计文档、代码审核报告 |
| **Dev Agent** | Story 设计、编码实现、审核挑战响应 | Dev Story、源代码 |
| **Test Agent** | 测试方案设计、流程验证 | 测试方案、验证报告 |

## 核心机制

### 挑战机制（最高优先级）

下游智能体发现上游产出违反规范/标准时，可发起正式挑战。挑战必须引用具体规范条款，被挑战方必须立即停止当前工作优先处理。代码审核阶段（Phase 6）的 8 项检查失败会自动触发系统生成型挑战。

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

**任一 FAIL → 自动挑战 → 修复 → 重审 → 全部 PASS 才能进入 Phase 7 验证。**

### 插件扩展机制

12 个 Hook 点分布在每个阶段转换处，允许项目注入自定义行为（校验规则、自动化脚本、补充文档等）。

详见: `plugins/README.md`

## 编码规范（预置 4 种语言）

| 语言 | 文件 | 章节 |
|------|------|------|
| JavaScript (ES2022+) | `framework/standards/coding-standards.javascript.md` | 15 章 + 命名速查表 + 禁止项清单 |
| TypeScript (5.x+) | `framework/standards/coding-standards.typescript.md` | 15 章 + 命名速查表 + 禁止项清单 |
| Java (17 LTS+) | `framework/standards/coding-standards.java.md` | 15 章 + 命名速查表 + 禁止项清单 |
| Python (3.11+) | `framework/standards/coding-standards.python.md` | 15 章 + 命名速查表 + 禁止项清单 |

其他语言在初始化时从通用模板 `framework/standards/coding-standards.template.md` 自动生成。

## 目录结构

```
H5LifecycleTemplate/
├── CLAUDE.md                              # 主编排器（框架入口）
├── Prompt.md                              # 原始架构设计文档
├── README.md                              # 本文件
├── .claude/settings.json                  # Claude Code 配置
├── framework/
│   ├── agents/                            # 智能体技能定义
│   │   ├── se-agent.skill.md
│   │   ├── dev-agent.skill.md
│   │   └── test-agent.skill.md
│   ├── workflows/                         # 工作流规则
│   │   ├── standard-lifecycle.rule.md     # 标准生命周期（7 阶段 + 快捷入口）
│   │   ├── challenge-mechanism.rule.md    # 挑战/纠偏机制
│   │   └── plugin-extension.rule.md       # 插件扩展机制
│   ├── artifacts/                         # 制品模板
│   │   ├── prd.template.md
│   │   ├── se-design.template.md
│   │   ├── dev-story.template.md
│   │   ├── test-plan.template.md
│   │   └── architecture-doc.template.md
│   └── standards/                         # 规范文档
│       ├── coding-standards.template.md   # 通用编码规范模板
│       ├── coding-standards.javascript.md
│       ├── coding-standards.typescript.md
│       ├── coding-standards.java.md
│       ├── coding-standards.python.md
│       ├── project-structure.template.md
│       └── validation-standards.template.md
├── plugins/                               # 插件扩展
│   ├── README.md
│   └── example-plugin/
└── docs/                                  # 生成文档（初始化时产出）
    ├── .framework-init.lock
    ├── architecture.md
    ├── module-map.md
    ├── coding-standards.md
    └── project-structure.md
```

## 设计原则

1. **模板驱动** — 所有智能体产出遵循统一模板，标准违背可被机械检测
2. **挑战优先** — 纠偏机制优先级最高，防止错误级联
3. **Hook 注入** — 12 个插件 Hook 点覆盖每个阶段转换，不修改核心流程即可扩展
4. **文档跟踪状态** — 通过 `docs/` 下文件存在性隐式跟踪生命周期进度，无需额外状态存储
5. **语言无关内核** — 核心工作流通用，语言特定规范独立管理，初始化时按需加载
6. **强制初始化** — 所有生命周期操作前检查锁文件，确保项目信息完整准确
