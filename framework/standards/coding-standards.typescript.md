# TypeScript 编码规范

## 文档元数据
- **语言:** TypeScript
- **版本:** 5.x+
- **运行环境:** Node.js / Browser
- **参考标准:** Airbnb JavaScript Style Guide, TypeScript ESLint, Google TypeScript Style Guide

---

## 0. 项目检测约定（扫描驱动）

本章节由 Scanner Agent 在 Phase 0 初始化时自动生成，记录从实际代码中检测到的项目特有约定。检测约定 **优先于** 后续通用规则 — 当两者冲突时，以本章节为准。

### 0.1 检测机制
Scanner Agent 在 Phase F 对全量代码进行采样分析（每个模块最多 20 个文件），统计每种约定的覆盖率。覆盖率 >70% 的模式被认定为 **主导约定**。

标记含义：
- ✓ **confirmed** — 项目代码与通用标准一致，通用规则保持
- ⚠ **overridden** — 项目代码与通用标准不同，项目约定覆盖通用规则
- ◈ **generic** — 通用规则，无法由扫描检测（如安全规范），保持生效
- ⬢ **remediation** — 覆盖率 <70%，存在不一致，需整改

### 0.2 公共方法约定（Scanner Phase F.1）
| 约定项 | 检测结果 | 覆盖率 | 标记 | 说明 |
|--------|---------|--------|------|------|
| 命名风格 | `{待扫描填充}` | — | — | — |
| 动词前缀 | `{待扫描填充}` | — | — | — |
| 异步模式 | `{待扫描填充}` | — | — | — |
| 错误处理 | `{待扫描填充}` | — | — | — |
| 参数命名 | `{待扫描填充}` | — | — | — |

### 0.3 文件组织约定（Scanner Phase F.2）
| 约定项 | 检测结果 | 覆盖率 | 标记 | 不一致文件 |
|--------|---------|--------|------|-----------|
| 目录命名 | `{待扫描填充}` | — | — | — |
| 组件文件命名 | `{待扫描填充}` | — | — | — |
| 工具文件命名 | `{待扫描填充}` | — | — | — |
| 测试文件命名 | `{待扫描填充}` | — | — | — |
| 桶导出使用 | `{待扫描填充}` | — | — | — |
| Feature 结构 | `{待扫描填充}` | — | — | — |

### 0.4 编码风格约定（Scanner Phase F.3）
| 约定项 | 检测结果 | 覆盖率 | 标记 | 偏离文件 |
|--------|---------|--------|------|-----------|
| 缩进 | `{待扫描填充}` | — | — | — |
| 引号风格 | `{待扫描填充}` | — | — | — |
| 分号 | `{待扫描填充}` | — | — | — |
| 尾逗号 | `{待扫描填充}` | — | — | — |
| 函数风格 | `{待扫描填充}` | — | — | — |
| 返回类型 | `{待扫描填充}` | — | — | — |
| 导入排序 | `{待扫描填充}` | — | — | — |
| null/undefined | `{待扫描填充}` | — | — | — |
| 错误模式 | `{待扫描填充}` | — | — | — |
| 行宽限制 | `{待扫描填充}` | — | — | — |

### 0.5 冲突处理规则
1. **本章节有值且标记为 ⚠ overridden** → 以本章节约定为准，对应通用章节的规则被覆盖
2. **本章节标记为 ✓ confirmed** → 通用规则适用，本章节仅作确认记录
3. **本章节标记为 ◈ generic** → 通用规则适用，无法自动检测
4. **本章节标记为 ⬢ remediation** → 通用规则适用，但项目存在不一致需人工整改
5. **本章节值为 `{待扫描填充}`** → Scanner 尚未执行或语言不支持该检测项

---

### 1.1 文件命名
- **模式:** kebab-case（小写+连字符）
- **类型定义文件:** `*.d.ts`
- **React 组件文件:** PascalCase — `UserProfile.tsx`
- **测试文件:** `*.spec.ts` 或 `*.test.ts`
- **示例:** `user-service.ts`, `api-client.ts`, `date-utils.ts`

### 1.2 变量命名
- **局部变量:** camelCase — `let userName: string = 'Alice';`
- **常量:** UPPER_SNAKE_CASE — `const MAX_RETRY_COUNT = 3;`
- **枚举成员:** PascalCase — `Color.Red`
- **布尔值:** `is`/`has`/`should`/`can` 前缀 — `isLoading`, `hasError`

### 1.3 函数命名
- **模式:** camelCase，动词开头
- **类型守卫:** `is` + 类型名 — `isUser(data: unknown): data is User`
- **工厂函数:** `create` + 实体名 — `createUserService()`

```typescript
// 正确
function getUserById(id: string): Promise<User | null> { ... }
function validateEmail(email: string): ValidationResult { ... }
function isUser(data: unknown): data is User { ... }

// 错误
function user(id: string) { ... }     // 缺少动词、缺少返回类型
```

### 1.4 类型和接口命名
- **接口:** PascalCase，无 I 前缀 — `User`, `UserRepository`
- **类型别名:** PascalCase — `UserRole`, `ApiResponse<T>`
- **泛型参数:** 单字母大写或描述性 PascalCase — `T`, `TData`
- **枚举:** PascalCase — `OrderStatus`

```typescript
// 正确
interface User {
  id: string;
  name: string;
}

type UserRole = 'admin' | 'user' | 'guest';

enum OrderStatus {
  Pending = 'pending',
  Processing = 'processing',
  Completed = 'completed',
}

// 错误
interface IUser { ... }       // 不使用 I 前缀
type userRole = string;        // type 应使用 PascalCase
```

---

## 2. 类型系统

### 2.1 类型注解
- **函数返回类型：** 必须显式标注（公共 API）
- **函数参数类型：** 必须显式标注
- **变量类型：** 能推断时不标注，无法推断时标注
- **回调参数：** 能推断时省略

```typescript
// 正确：返回类型显式标注
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// 正确：变量类型可推断
const user = getUserById('123');        // 类型可推断
const items: Item[] = [];                // 空数组无法推断

// 正确：回调参数类型可推断
items.filter(item => item.price > 100);  // item 可推断

// 错误：缺少返回类型
function calculateTotal(items: Item[]) { ... }
```

### 2.2 类型 vs 接口
- **接口（interface）** — 优先使用，用于定义对象结构
- **类型别名（type）** — 用于联合类型、交叉类型、元组
- **选择原则:** 能用 interface 就用 interface

```typescript
// 优先：interface
interface User {
  id: string;
  name: string;
}

// 适用场景：联合类型
type Status = 'idle' | 'loading' | 'success' | 'error';

// 适用场景：组合类型
type UserWithPosts = User & { posts: Post[] };
```

### 2.3 泛型
- 使用描述性名称（简短泛型可用 `T`, `K`, `V`）
- 添加约束条件
- 避免过度泛型化

```typescript
// 正确：清晰的泛型命名和约束
interface Repository<TEntity extends { id: string }> {
  findById(id: string): Promise<TEntity | null>;
  save(entity: TEntity): Promise<TEntity>;
  delete(id: string): Promise<void>;
}

// 错误：无约束、泛型过多
function process<T, U, V, W>(a: T, b: U, c: V): W { ... }
```

### 2.4 空值处理
- 启用 `strictNullChecks`
- 使用可选链 `?.` 和空值合并 `??`
- 明确标注可空类型: `string | null` 或 `string | undefined`
- 避免使用非空断言 `!` 除非绝对确定

```typescript
// 正确
const name = user?.profile?.name ?? 'Anonymous';
function findUser(id: string): User | null { ... }

// 错误：滥用非空断言
const name = user!.profile!.name!;
```

### 2.5 类型断言
- 使用 `as` 语法，不使用尖括号语法
- 仅在必要时使用（边界转换、测试 mock）
- 避免无依据的类型断言

```typescript
// 正确
const element = document.getElementById('app') as HTMLDivElement;

// 错误
const element = <HTMLDivElement>document.getElementById('app');  // 与 JSX 冲突
const value = data as any as User;  // 任意类型链式断言
```

---

## 3. 代码结构

### 3.1 文件组织
```
1. 导入语句（分组：标准库 → 第三方 → 内部模块 → 类型导入）
2. 类型/接口定义
3. 常量
4. 模块级变量
5. 函数/类定义
6. 导出
```

```typescript
// 第三方
import express from 'express';
import { z } from 'zod';

// 内部模块
import { UserService } from './services/user-service.js';
import { formatDate } from './utils/date-utils.js';

// 类型导入（使用 type 关键字）
import type { User, CreateUserInput } from './types/user.js';

// 常量
const DEFAULT_PAGE_SIZE = 20;

// 导出函数
export async function createUser(input: CreateUserInput): Promise<User> { ... }
```

### 3.2 函数规范
- **最大参数:** 3 个（超出使用对象参数）
- **单一职责:** 每个函数只做一件事
- **尽早返回:** 减少嵌套
- **避免布尔参数:** 使用选项对象或拆分为两个函数

```typescript
// 正确：选项对象代替布尔参数
function searchUsers(query: string, options?: SearchOptions): Promise<User[]> { ... }

interface SearchOptions {
  includeInactive?: boolean;
  page?: number;
  limit?: number;
}

// 错误
function searchUsers(query: string, includeInactive: boolean): Promise<User[]> { ... }
```

---

## 4. 格式化

### 4.1 缩进与行长
- **缩进:** 2 个空格
- **最大行长:** 100 字符

### 4.2 大括号
- **风格:** 1TBS
- 所有控制语句必须使用大括号

### 4.3 引号
- **字符串:** 单引号
- **模板字符串:** 反引号（需要变量插值时）

### 4.4 分号
- **必须使用分号**

### 4.5 尾逗号
- 多行对象/数组/参数使用尾逗号

```typescript
const user: User = {
  id: '123',
  name: 'Alice',
  email: 'alice@example.com',  // 尾逗号
};
```

---

## 5. 类与面向对象

### 5.1 类成员声明
```typescript
class UserService {
  // 1. 静态属性
  private static instance: UserService | null = null;

  // 2. 实例属性（按访问修饰符排序：public → protected → private）
  private readonly repository: UserRepository;
  private cache: Map<string, User>;

  // 3. 构造函数
  public constructor(repository: UserRepository) {
    this.repository = repository;
    this.cache = new Map();
  }

  // 4. 公共方法
  public async findById(id: string): Promise<User | null> { ... }

  // 5. 私有方法
  private async fetchFromCache(id: string): Promise<User | null> { ... }
}
```

### 5.2 访问修饰符
- 明确标注 `public`、`protected`、`private`
- 默认值 `public` 也应显式标注
- 优先使用 `readonly`

### 5.3 组合优于继承
```typescript
// 正确：使用组合
class UserNotifier {
  constructor(
    private readonly emailService: EmailService,
    private readonly smsService: SmsService,
  ) {}

  async notify(user: User, message: string): Promise<void> {
    await this.emailService.send(user.email, message);
  }
}
```

---

## 6. 枚举与常量

### 6.1 字符串枚举（推荐）
```typescript
// 推荐：字符串枚举
enum OrderStatus {
  Pending = 'pending',
  Processing = 'processing',
  Completed = 'completed',
  Cancelled = 'cancelled',
}
```

### 6.2 const 断言（替代数字枚举）
```typescript
// 轻量替代方案
const ORDER_STATUS = {
  Pending: 'pending',
  Processing: 'processing',
  Completed: 'completed',
} as const;

type OrderStatus = (typeof ORDER_STATUS)[keyof typeof ORDER_STATUS]; // 'pending' | 'processing' | 'completed'
```

---

## 7. 模块与导入

### 7.1 导入类型
- 使用 `import type` 进行仅类型导入
- 类型和值的导入分开

```typescript
import type { User, UserRole } from './types/user.js';
import { UserService } from './services/user-service.js';
```

### 7.2 桶导出
- `index.ts` 仅用于重新导出，不包含逻辑
- 使用 `export *` 需要明确限制导出范围

```typescript
// index.ts — 仅作为桶导出
export { UserService } from './user-service.js';
export { AuthService } from './auth-service.js';
export type { User, CreateUserInput } from './types.js';
```

### 7.3 路径别名
- 使用 tsconfig 路径别名替代深层相对路径

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"],
      "@shared/*": ["./src/shared/*"],
      "@features/*": ["./src/features/*"]
    }
  }
}
```

```typescript
// 正确
import { UserService } from '@features/user/user-service.js';

// 避免
import { UserService } from '../../../features/user/user-service.js';
```

### 7.4 模块组织（扫描驱动）

项目初始化时，Scanner Agent 扫描全量代码并生成 `docs/module-map.md` 和 `docs/.scanner-report.json`。以下规则由扫描结果驱动，所有新增代码必须遵守。

#### 7.4.1 组织模式遵循
- **扫描检测到的组织模式**（feature-based / layer-based / hybrid）必须在整个项目中保持一致
- 新增模块必须遵循已检测到的目录结构模式
- 如需切换组织模式，必须先通过 SE Agent 设计迁移方案并更新 `docs/project-structure.md`

```typescript
// Feature-based 项目（扫描检测到 src/features/ 结构）
// 正确：新功能放入独立 feature 目录
src/features/payment/
├── components/
├── services/
├── types/
└── index.ts

// 错误：在 feature-based 项目中创建顶层 layer 目录
src/services/payment-service.ts  // 破坏了 feature-based 模式
```

#### 7.4.2 共享组件放置
- **Scanner 标记为 Shared 的组件**必须保留在共享目录中（如 `src/shared/components/`）
- 新增的跨模块组件（被 2+ 模块使用）必须放入共享目录，**不得**放入任何单一 feature 目录
- 仅被单一模块使用的组件保持在该模块目录内

```typescript
// Button 被 auth、dashboard、settings 三个模块使用 → 共享组件
// 正确位置
src/shared/components/Button.tsx

// 错误位置
src/features/auth/components/Button.tsx  // 其他模块无法合理引用
```

#### 7.4.3 共享方法/工具函数放置
- **Scanner 标记为 Shared Method 的函数**必须保留在共享工具目录中（如 `src/shared/utils/`）
- 新增的被 2+ 模块引用的工具函数必须放入共享目录
- 单一模块专用的工具函数保持在模块内的 `utils/` 目录
- Scanner 标记为 Dead Code Candidate 的未引用导出应删除

```typescript
// formatDate 被 4 个模块使用 → 共享方法
// 正确位置
src/shared/utils/date.ts

// 仅在 auth 模块内使用 → 模块私有工具
// 正确位置
src/features/auth/utils/password-validator.ts
```

#### 7.4.4 模块依赖规则
Scanner 生成的依赖矩阵定义了模块间的合法依赖关系。新增代码不得违反以下规则：

| 规则 | 说明 | 严重级别 |
|------|------|----------|
| **Core → Business 禁止** | 入口/启动模块不得依赖业务模块 | ERROR |
| **Core → UI 禁止** | 入口/启动模块不得依赖 UI 模块 | ERROR |
| **Infrastructure → Business 禁止** | 基础设施模块不得依赖业务模块 | ERROR |
| **Utility → 任何项目模块 警告** | 工具模块应零依赖或仅依赖标准库 | WARNING |
| **Shared → Feature 禁止** | 共享组件/方法不得依赖任何 feature 模块 | ERROR |
| **Feature → Feature 直接引用 警告** | Feature 间直接引用应通过 shared 层 | WARNING |
| **循环依赖 禁止** | A→B→A 或 A→B→C→A | ERROR |

```typescript
// 违反 Shared → Feature 规则
// src/shared/components/DataTable.tsx
import { User } from '@features/user/types';  // ERROR: 共享组件依赖了 feature 模块

// 正确做法：将共享类型提取到 shared 层
import { User } from '@shared/types/user';
```

#### 7.4.5 路径别名使用
- 必须使用 `tsconfig.json` 中已配置的路径别名（由 Scanner 在 `docs/.scanner-report.json` 中记录）
- 禁止使用超过 2 层的相对路径（`../../`）
- 新增路径别名需要更新 `tsconfig.json` 并触发 Scanner 重新扫描

```typescript
// 正确：使用已注册的路径别名
import { Button } from '@shared/components/Button';
import { formatDate } from '@shared/utils/date';

// 错误：深层相对路径
import { Button } from '../../../shared/components/Button';
```

### 7.5 模块公共 API 管理

#### 7.5.1 桶导出规范
- 每个模块通过 `index.ts` 控制公共 API 面
- `index.ts` **仅包含 re-export 语句**，不得包含运行时逻辑
- 内部实现细节（未在 index.ts 中 re-export 的符号）不得被外部模块引用

```typescript
// src/features/auth/index.ts — 公共 API
export { AuthService } from './services/auth-service.js';
export { useAuth } from './hooks/use-auth.js';
export type { User, LoginInput } from './types.js';
// 注意：auth-utils.ts 未导出 → 模块私有，外部不得引用
```

#### 7.5.2 跨模块引用检查
- 引用其他模块时，只能通过其 `index.ts` 桶文件
- 禁止深路径导入其他模块的内部文件

```typescript
// 正确：通过桶文件
import { AuthService } from '@features/auth';

// 错误：深路径导入其他模块内部
import { hashPassword } from '@features/auth/utils/internal-crypto';
```

### 7.6 新增文件时的检查清单

添加新文件前，确认以下事项（参考 `docs/.scanner-report.json`）：

- [ ] 文件放置路径符合检测到的组织模式（§7.4.1）
- [ ] 若为组件：是否被多模块使用？→ 是则放 `shared/`，否则放模块内（§7.4.2）
- [ ] 若为工具函数：是否被多模块使用？→ 是则放 `shared/utils/`，否则放模块内（§7.4.3）
- [ ] 新增的 import 不违反模块依赖规则（§7.4.4）
- [ ] 使用已注册的路径别名，不使用深层相对路径（§7.4.5）
- [ ] 模块公共 API 通过桶文件控制（§7.5）
- [ ] 未引入新的循环依赖

---

## 8. 异步编程

### 8.1 async/await（唯一方式）
```typescript
async function getUserWithPosts(userId: string): Promise<UserWithPosts> {
  const user = await userRepository.findById(userId);
  if (!user) {
    throw new NotFoundError('User', userId);
  }
  const posts = await postRepository.findByUserId(userId);
  return { ...user, posts };
}
```

### 8.2 Promise 工具方法
```typescript
// 并行请求
const [user, settings] = await Promise.all([
  fetchUser(userId),
  fetchSettings(userId),
]);

// 容错并行（全部完成，不论成败）
const results = await Promise.allSettled([
  fetchFromCache(key),
  fetchFromDatabase(key),
]);

// 竞速
const result = await Promise.race([
  fetchWithTimeout(url, 5000),
  timeout(5000),
]);
```

---

## 9. 错误处理

### 9.1 自定义错误类
```typescript
abstract class AppError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;

  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  readonly code = 'NOT_FOUND';
  readonly statusCode = 404;

  constructor(resource: string, id: string) {
    super(`${resource} with id '${id}' not found`);
  }
}

class ValidationError extends AppError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 400;

  constructor(
    message: string,
    public readonly errors: FieldError[],
  ) {
    super(message);
  }
}
```

### 9.2 错误处理模式
- 使用 Result 类型模式（推荐用于复杂业务逻辑）
- 在边界层统一处理错误

```typescript
// Result 类型模式
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

async function createUser(input: CreateUserInput): Promise<Result<User, ValidationError>> {
  const validation = validateInput(input);
  if (!validation.valid) {
    return { success: false, error: new ValidationError('Invalid input', validation.errors) };
  }
  const user = await userRepository.save(input);
  return { success: true, data: user };
}
```

---

## 10. 高级类型模式

### 10.1 判别联合（Discriminated Unions）
```typescript
// 正确：使用判别联合处理不同状态
type RequestState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };

function renderState<T>(state: RequestState<T>): string {
  switch (state.status) {
    case 'idle': return 'Ready';
    case 'loading': return 'Loading...';
    case 'success': return `Data: ${state.data}`;    // 类型收窄
    case 'error': return `Error: ${state.error.message}`;
  }
}
```

### 10.2 模板字面量类型
```typescript
type EventName = `user:${'created' | 'updated' | 'deleted'}`;
// 'user:created' | 'user:updated' | 'user:deleted'

type ApiEndpoint = `/api/${string}`;
```

### 10.3 工具类型
```typescript
// 常用内置工具类型
type PartialUser = Partial<User>;           // 所有属性可选
type RequiredUser = Required<User>;         // 所有属性必填
type ReadonlyUser = Readonly<User>;         // 所有属性只读
type UserEmail = Pick<User, 'id' | 'email'>; // 选择部分属性
type UserWithoutPassword = Omit<User, 'password'>; // 排除部分属性
type UserRecord = Record<string, User>;     // 键值映射
```

### 10.4 satisfies 运算符
```typescript
// 保持字面量类型的同时确保符合接口
const config = {
  host: 'localhost',
  port: 3000,
  debug: true,
} satisfies AppConfig;
// typeof config.port 仍然是 number (3000) 而非 number
```

---

## 11. 注释

### 11.1 原则
- 不注释代码做了什么（命名已经说明）
- 注释为什么（非显而易见的决策、约束、算法选择）
- 不保留注释掉的代码

### 11.2 TSDoc
```typescript
/**
 * 创建新用户并发送欢迎邮件。
 * 如果邮箱已被注册，抛出 {@link DuplicateEmailError}。
 */
export async function registerUser(input: RegisterUserInput): Promise<User> {
  ...
}
```

---

## 12. 相等与比较

- 使用 `===` 和 `!==`
- 对象比较使用自定义相等函数或深度比较
- 字符串比较区分大小写时注意地区差异

---

## 13. 性能规范

- 避免 `as any` 断言（绕过类型检查）
- 避免在循环中创建复杂对象
- 大数据集使用 `Map`/`Set` 代替 `Object`/`Array` 查找
- 异步操作能并行的不串行
- 利用 `const` 断言减少类型推断开销

---

## 14. 安全规范

### 14.1 注入防护
- 禁止 `eval()`, `new Function()`, `setTimeout(string)`
- SQL: 使用参数化查询或 ORM
- 使用类型系统防止不安全的字符串拼接

### 14.2 输入验证
- 运行时验证所有外部输入（使用 zod、yup、io-ts 等）
- 编译时类型不足以确保运行时安全

```typescript
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional(),
});

type CreateUserInput = z.infer<typeof CreateUserSchema>;

function handleRequest(body: unknown): User {
  const input = CreateUserSchema.parse(body);  // 运行时验证
  return createUser(input);
}
```

---

## 15. 工具链配置

### 15.1 tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": false,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "sourceMap": true,
    "outDir": "./dist",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 15.2 ESLint (typescript-eslint)
```json
{
  "parser": "@typescript-eslint/parser",
  "plugins": ["@typescript-eslint"],
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/strict-type-checked",
    "plugin:@typescript-eslint/stylistic-type-checked"
  ],
  "rules": {
    "@typescript-eslint/explicit-function-return-type": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/prefer-optional-chain": "error",
    "@typescript-eslint/prefer-nullish-coalescing": "error",
    "@typescript-eslint/consistent-type-imports": "error"
  }
}
```

### 15.3 Prettier
```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "all",
  "printWidth": 100
}
```

---

## 附录A: 命名速查表

| 类型 | 规则 | 示例 |
|------|------|------|
| 文件 | kebab-case | `user-service.ts` |
| React 组件文件 | PascalCase | `UserProfile.tsx` |
| 变量 | camelCase | `userName` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| 函数 | camelCase（动词开头） | `getUserById()` |
| 类 | PascalCase | `UserService` |
| 接口 | PascalCase（无 I 前缀） | `User` |
| 类型别名 | PascalCase | `UserRole` |
| 枚举 | PascalCase | `OrderStatus` |
| 枚举成员 | PascalCase | `OrderStatus.Pending` |
| 泛型参数 | 单字母或 PascalCase | `T`, `TData` |
| 布尔值 | is/has/should/can | `isLoading` |
| 私有字段 | #camelCase 或 private camelCase | `#cache` |

## 附录B: 禁止事项

- [ ] 使用 `any`
- [ ] 使用 `as any` 强制转换
- [ ] 使用非空断言 `!`
- [ ] 使用 `var`
- [ ] 缺少函数返回类型（公共 API）
- [ ] 使用 `I` 前缀命名接口
- [ ] 空的 catch 块
- [ ] 使用 `eval()` / `new Function()`
- [ ] 硬编码密钥
- [ ] 循环依赖
- [ ] 导出可变全局状态
