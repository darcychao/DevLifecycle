# UX Constraint Enforcement (UX约束强制机制)

## Overview

When a project has a UI component (detected during Phase 0 initialization and recorded in `docs/.framework-init.lock` → `ui_stack.has_ui = true`), PRD and Dev Story design phases that involve UI design MUST include user-provided UX constraints. This rule enforces that requirement and blocks phase progression until UX constraints are satisfied.

## Applicability

UX constraint enforcement activates when ALL of these conditions are true:
1. `docs/.framework-init.lock` contains `ui_stack.has_ui: true`
2. The current phase is Phase 1 (PRD) or Phase 3 (Story Design)
3. The requirement being worked on involves UI design (user-facing interfaces, visual components, interaction flows)

**Determining if a requirement involves UI design:**
- PRD contains user stories with visual/interaction components (forms, pages, dashboards, etc.)
- PRD references UI elements (buttons, inputs, navigation, layouts, modals, etc.)
- Dev Story tasks involve creating or modifying UI components
- SE Design specifies frontend modules or UI components

## Mandatory UX Constraints

When UX constraint enforcement is active, the user MUST provide the following in the relevant artifact:

### For PRD (Phase 1)

| Constraint ID | Category | Required Information | Blocking |
|--------------|----------|---------------------|----------|
| UX-01 | Interaction Flow | 用户操作路径和页面导航流程（页面跳转图、用户操作步骤说明） | Yes |
| UX-02 | Visual Reference | 视觉参考或设计稿（Figma/Sketch 链接、线框图、或现有页面样式参考） | Yes |
| UX-03 | Responsive Design | 响应式设计要求（支持的设备类型、断点值、不同屏幕尺寸的布局差异说明） | Yes (mobile/web projects) |
| UX-04 | Form Interaction | 表单交互规范（字段验证规则、错误提示文案、提交行为、成功/失败反馈） | Yes (if forms exist) |
| UX-05 | Feedback States | 状态反馈说明（加载中、空状态、错误状态、边界情况的 UI 表现） | Yes |
| UX-06 | Accessibility | 可访问性要求（键盘导航、屏幕阅读器支持、色彩对比度、i18n需求） | No (WARNING) |
| UX-07 | Animation/Transition | 动效与过渡要求（页面切换动画、交互反馈动效、是否需要遵循系统"减少动效"设置） | No (WARNING) |

### For Dev Story (Phase 3)

| Constraint ID | Category | Required Information | Blocking |
|--------------|----------|---------------------|----------|
| UX-S-01 | Component Specification | UI 组件的视觉规格（尺寸、间距、颜色、字体、状态变体） | Yes |
| UX-S-02 | Component Behavior | 组件交互行为说明（hover/active/focus/disabled 状态、点击/拖拽/滚动响应） | Yes |
| UX-S-03 | Layout Implementation | 布局实现方式（使用的布局组件/系统、响应式断点实现方案） | Yes |
| UX-S-04 | Styling Method | 样式实现方式（Tailwind 类名/CSS Module/styled-component，需与项目 §0.6 一致） | Yes |
| UX-S-05 | Accessibility Implementation | 无障碍实现方案（ARIA 属性、键盘导航、焦点管理、语义化 HTML） | No (WARNING) |

## Enforcement Protocol

### Phase 1 — PRD UX Check

```
User submits PRD
       │
       ▼
Orchestrator checks: Does this PRD involve UI design?
       │
       ├── NO → UX constraints not required → proceed normally
       │
       └── YES → Check UX-01 through UX-07
                    │
                    ├── ALL blocking constraints (UX-01 to UX-05) satisfied → Pass → proceed to Phase 2
                    │
                    ├── WARNING constraints missing → Warn user but proceed
                    │
                    └── BLOCKING constraint missing → STOP
                         Orchestrator queries user for missing constraints:
                         "本 PRD 涉及 UI 设计，缺少以下 UX 约束信息：
                          - UX-01: 交互流程说明（页面导航图、用户操作路径）
                          - UX-03: 响应式设计要求
                          请补充以上信息后继续。"
```

### Phase 3 — Story UX Check

```
Dev Agent begins Story Design (Phase 3)
       │
       ▼
Orchestrator checks: Does this Dev Story involve UI implementation?
       │
       ├── NO → UX constraints not required → proceed normally
       │
       └── YES → Check UX-S-01 through UX-S-05
                    │
                    ├── ALL blocking constraints (UX-S-01 to UX-S-04) satisfied → Pass → proceed to Phase 4
                    │
                    ├── WARNING constraints missing → Warn Dev Agent but proceed
                    │
                    └── BLOCKING constraint missing → STOP
                         Orchestrator blocks Dev Story submission:
                         "本 Dev Story 涉及 UI 实现，缺少以下 UX 规格信息：
                          - UX-S-01: 组件视觉规格
                          - UX-S-03: 布局实现方式
                          请 Dev Agent 与用户确认后补充。"
```

### UX Constraint Pre-Check (Phase 2 → Phase 3)

Before Phase 3 begins, the SE Agent (as reviewer of PRD) checks:
- If PRD involves UI design AND UX constraints are incomplete → **raise CAT-1 challenge** against PRD
- Challenge basis: "PRD requires UI design but missing UX constraints [list missing IDs]"

### UX Constraint Post-Check (during Code Review)

During Phase 6 Code Review, the SE Agent checks (CR-2: Dev Story Alignment extension):
- If Dev Story included UI tasks AND UX-S constraints were followed
- If UI implementation deviates from UX constraints without documented justification → fail CR-2

## Integration with Existing Templates

### PRD Template Addition

PRD templates include the UX Constraints section at `§5.4 UX Constraints` within `§5. Constraints & Assumptions`. See `framework/artifacts/prd.template.md`.

### Dev Story Template Additions

Dev Story templates include:
- UX specifications within each UI-related TASK's implementation details
- §6 Standards Compliance Checklist extended with UX compliance items

### SE Design Template Additions

SE Design templates include UX architecture considerations in §3 Module Design for UI modules and §4.4 State Management (UI state).

## UX Constraint in Lock File

The `ui_stack` in `docs/.framework-init.lock` indicates UX constraint applicability:

```json
{
  "ui_stack": {
    "has_ui": true,
    "ui_framework": "React 18.x",
    "component_library": "@mui/material 5.x",
    "styling": ["tailwindcss 3.x"],
    "ux_constraints_required": true
  }
}
```

If `ux_constraints_required` is `true`, the UX constraint enforcement mechanism is active for all phases involving UI design.

## Exception Handling

1. **Purely backend API project with admin UI dependency**: If UI is present but the current requirement is backend-only, UX constraints are waived for that requirement.
2. **Incremental UI change with no new visual design**: If the Dev Story only modifies existing UI logic (no new visual elements), UX-S-01 may be waived with explicit documented justification.
3. **User explicitly declares no UX input available**: UX-02 may be waived, but UX-01 and UX-05 remain mandatory. The user's declaration is recorded in the PRD.
4. **Re-initialization**: If the project structure changes to remove UI, update `ui_stack.has_ui` to `false` in the lock file.
